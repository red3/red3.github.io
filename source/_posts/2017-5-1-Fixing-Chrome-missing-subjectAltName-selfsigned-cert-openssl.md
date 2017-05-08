---
layout: post
title:  "Fixing Chrome 58+ [missing_subjectAltName] error when using self signed certificates"
date:   2017-5-1 18:30:00
---


最近在开发 [PlayAppStore](https://github.com/playappstore/PlayAppStore) 项目的时候，发现一个问题， 谷歌 Chrome 浏览器从 58 版本开始，不再信任缺少 **SAN** (Subject Alternative Name)的证书，具体来说，就是在 PlayAppStore 项目中，我们使用了自签名的证书来提供 HTTPS 服务，这个自签名证书仅包含常用的 **CN**(Common Name) 字段。

所以，自新版本开始，当用户通过谷歌浏览器访问使用了这个证书的网站的时候，就会报这样的错：

![error](http://photo-coder.b0.upaiyun.com/img/missing_subjectAltName01.png)

虽然仅仅是差了一个字段，但是解决起来，也是费了一番功夫。

### 自签名证书情况

如果说，仅仅是给自签名证书加入这个 **SAN** 字段，还好说，通过 `openssl` 命令增加 `config` 参数就可以解决问题，见下： 

```
openssl req \
    -newkey rsa:2048 \
    -x509 \
    -nodes \
    -keyout server.key \
    -new \
    -out server.pem \
    -subj /CN=localhost \
    -reqexts SAN \
    -config <(cat /System/Library/OpenSSL/openssl.cnf \
        <(printf '[SAN]\nsubjectAltName=DNS:localhost')) \
    -sha256 \
    -days 3650
```

> 注：上面的解决方案来自 github 讨论，通过这个[链接](https://github.com/webpack/webpack-dev-server/issues/854)了解更多内容

具体来说就是 SAN 字段无法在命令行中直接配置，需要在配置文件中指定，故需要你配置文件中增加 `subjectAltName=DNS:localhost` 来使 SAN 的配置生效，这里需要注意的是，上面的配置文件路径适用于 MacOS 用户，其他系统用户需要替换为自己主机上的具体地址。

### 从自签名 CA 请求证书情况

但是如果你配置了自己的 CA，需要这个 CA 来签发证书的话，情况就有点不一样了。
首先是先创建证书请求文件：

```
openssl genrsa \
  -out private.key \
  2048 \
  
openssl req \
  -new \
  -nodes \
  -key private.key \
  -out csr.req \
  -reqexts SAN \
  -config <(cat /System/Library/OpenSSL/openssl.cnf \
        <(printf '[SAN]\nsubjectAltName=DNS:localhost')) \
  -subj "/C=US/ST=Utah/L=Provo/O=PLAYAPPSTORE Tech Inc/CN=localhost" 
  
```

对比之前的情况就可以发现，这次我们缺少了 `x509` 字段，并且产出的是 	`csr.req`，因为我们是为了创建证书请求文件，并不是为了生成证书。我们在证书请求文件中包好了 SAN 字段，然后拿这个请求文件去 CA 那里请求 CA 给颁发证书。

为了验证是否成功包含 SAN 字段，我们可以用下面的命令，看输出中是否有包含设定的 DNS.

```
openssl req -text -in csr.req -noout
```

在 CA 颁发证书这个环节，之前的时候，一般是通过下面的方式：

```
openssl x509 \
  -req -in csr.req \
  -CA my-root-ca.cer \
  -CAkey my-root-ca.key \
  -CAcreateserial \
  -sha256 \
  -out cert.cer \
  -days 365
```

但是问题就是出现在了这个环节，通过下面的方式验证生成的证书并没有包含请求文件中的 SAN 字段：

```
openssl x509 -text -in cert.cer -noout
```
并且奇怪的是，用上面的命令生成的证书的版本竟然还是 V1 (见输出中的字段中有 Version: 1)，而我们期望生成的应该是 V3 证书才对。

带着这个好奇，我查看了 MacOS 系统下自带的 openssl 的配置文件，发现了下面的说明：

> To use this configuration file with the "-extfile" option of the
 openssl x509 utility.  name here the section containing the X.509v3 extensions to use `extensions    =`

意思就是说当使用 openssl x509 命令生成证书的时候，想要包含 X.509v3 拓展的话，需要使用 extfile 参数来指定这个配置文件，并且指定具体的 extension. 遗憾的是，默认的配置文件并没有指定具体的 extensions, 不过我们可以自己生成自己的 extensions：



```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = localhost
IP.1 = 127.0.0.1

```

我们把上面的配置文件保存为 v3.cnf, 然后重新修改签名证书的命令：

```
openssl x509 \
  -req -in csr.req \
  -CA my-root-ca.cer \
  -CAkey my-root-ca.key \
  -CAcreateserial \
  -sha256 \
  -out cert.cer \
  -extfile v3.cnf \
  -days 375
  
```

再次验证证书的时候，发现证书已经是 V3 格式了，并且在 X509v3 Subject Alternative Name 栏目下也是可以看见我们刚才添加的 SAN 内容：

```
DNS:localhost, IP Address:127.0.0.1
```



大功告成，满心欢喜以为解决了问题，过了一会，突然隐隐觉得哪有不对：命名已经在证书请求文件中设置过 SAN 了，为什么在签发证书的时候在重新再搞一次？想想这样一个问题，你向正规的 CA 机构请求证书，在请求文件中设置了自己的 SAN, 但是 CA 机构忽略了你的 SAN 内容，替换成了别的内容，这样肯定是有问题的啊! 正经的做法应该是读取请求文件中的 SAN 才对。

再一次，看看默认的配置文件，发现了下面的配置：

```
[ CA_default ]
 # Extension copying option: use with caution.
 # copy_extensions = copy
```

这个配置默认是关闭的，从字面上来看，和我们的需求相符合，就是拷贝证书拓展内容，但是注释中说要小心使用这个配置，究竟要小心什么，并没有详细说明，只能借助文档了。

从 openssl 的[官方文档](https://www.openssl.org/docs/manmaster/man1/ca.html)中找到了这个字段的说明：

```
copy_extensions
Determines how extensions in certificate requests should be handled. If set to none or this option is not present then extensions are ignored and not copied to the certificate. If set to copy then any extensions present in the request that are not already present are copied to the certificate. If set to copyall then all extensions in the request are copied to the certificate: if the extension is already present in the certificate it is deleted first. See the WARNINGS section before using this option.

The main use of this option is to allow a certificate request to supply values for certain extensions such as subjectAltName.
```

我尤其注意到下面这句话：

> The main use of this option is to allow a certificate request to supply values for certain extensions such as subjectAltName.

Bingo, 文档中着重解释了这个字段的主要作用就是用来处理证书请求文件中的 subjectAltName 拓展的，看来我们找对了方向。

但是 `copy_extensions = copy` 配置是位于 `openssl ca` 命令下的，意味着我们要抛弃之前使用的 `openssl x509` 改用这个命令。现在思路是这样，修改或者新建配置文件，使 `copy_extensions = copy` 生效:

```
[ ca ]
default_ca      = CA_default
[ CA_default ]
dir            = ./demeCA
database       = $dir/index.txt
serial         = $dir/serial
RANDFILE       = $dir/private/.rand
new_certs_dir  = $dir/newcerts
copy_extensions = copy
policy         = policy_any
x509_extensions = usr_cert
[ policy_any ]
countryName            = supplied
stateOrProvinceName    = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional
[ usr_cert ]
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment

```

这里需要注意的是，CA_default 里面需要指定特定的文件夹，如果你要使用自己新建的配置文件，要确保配置文件中指定的文件及文件夹确实存在，有了这个配置文件，我们就可以重新对针对证书请求文件颁发证书了 (假定这个配置文件保存为 ca.cnf)：

```
openssl ca \
-config ca.cnf \
-keyfile my-root-ca.key \
-cert my-root-ca.cer \
-days 375  \
-md sha256 \
-in csr.req \
-out cert.cer \
-batch 
```

### Use with caution

之前说过，默认的配置文件中指出使用 `copy_extensions` 配置要加以小心，具体小心什么，配置文件中并没有说，但是从官方文档中可以了解到，只所以要小心，是因为拷贝拓展的时候，可能把不安全的配置也一并拷贝进去。举例来说，如果一个证书请求文件中误把 basicConstraints 设为 CA:TRUE 并且配置文件中把 copy_extensions 设为了 copyall, 这样就会签发出一个有效的 CA 证书。

如何避免这样的情形发生？那就是使用中间证书，因为 CA 根证书的保密措施，不可能签发证书的时候频繁使用根证书去签发，相反，是先用根证书签发出一个中间证书，后续的签发工作则有中间证书接管，这样，如果中间证书的密钥泄露了，那么根证书只要吊销这个中间证书就行了。话说回来，用中间证书来签发的时候，是怎么杜绝上述不安全情形的？诀窍就在下面这行配置：

```
basicConstraints = CA:TRUE, pathlen:0
```

这个配置通过 pathlen:0 明确指出该中间证书下面不会再有 CA 证书，故即使证书请求文件中出现 CA:TRUE 也会被忽略。



