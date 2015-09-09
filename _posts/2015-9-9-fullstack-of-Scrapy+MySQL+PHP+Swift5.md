---
layout: post
title:  "Scrapy+MySQL+PHP+Swift开发攻略系列（四）之爬虫被封+爬虫自动运行篇"
date:   2015-09-08 11:11:11
categories: jekyll update
tags:
  - Linux
  - Scrapy
  - Swift
---

##系列目录

你可以从这个地方做一个快速跳转。

- [Scrapy+MySQL+PHP+Swift开发攻略系列（一）之前言篇](http://blog.coderharry.com/2015/08/08/fullstack-of-Scrapy+MySQL+PHP+Swift1.html)
- [Scrapy+MySQL+PHP+Swift开发攻略系列（二）之爬虫篇](http://blog.coderharry.com/2015/08/08/fullstack-of-Scrapy+MySQL+PHP+Swift2.html)
- [Scrapy+MySQL+PHP+Swift开发攻略系列（三）之数据库MySQL篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（四）之爬虫被封+爬虫自动运行篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（五）之API篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（六）之RESTAPI篇]()
- [Scrapy+MySQL+PHP+Swift开发攻略系列（七）之Swift篇]()

## 温故知新

在前面的系列中，我们编写了爬虫，并且设置爬虫在每天的固定时间运行，对于频繁的爬虫请求，设置了随机的`User-Agent`，到现在为止，我们的数据库中应该已经存储了大量的数据，如图：

![](/assets/2015/fullstack_api01.png)

此次系列我们要编写`PHP`程序，提供客户端跟后台通信的接口。

## 准备工作

开始之前，确保机器上已经装有Apache、MySQL和PHP。好在Mac上已经自带Apache和PHP了，之前的系列中我们也在机器上安装了MySQL。要启用Apache，在终端执行下面的命令：

	sudo apachectl start
	
Mac上的Apache默认WWW目录是在`/Library/WebServer/Documents`，如果有需要，可以把默认WWW目录改成其他目录。

Apache默认是没有启用PHP支持的，要开启的话，应该这样：
	
	sudo vi /etc/apache2/httpd.conf
	// 找到下面这行，将前面的#去掉
	# LoadModule php5_module libexec/apache2/libphp5.so
	// 重启Apache
	sudo apachectl start

## 创建PHP工程

环境搭好以后，我们需要测试一下是否服务器可以正常工作。在WWW目录下新建一个名为`test.php`的文件，写入下面的测试代码。然后通过在浏览器中访问`http://localhost//test.php`可以在窗口中看到"Hello, World"说明正常工作了。

	// test.php
	<?php
		echo "Hello, World";
	?>

## 通过PHP连接MySQL

现在需要一个PHP类来连接MySQL，这个类的主要目的就是打开或者关闭一个连接到数据库的连接。所以创建两个文件：config.db.php 和 database.class.php.

- config.db.php : 配置数据库连接的用户名、密码、端口等。
- database.class.php : 创建一个到数据库的连接

以下为这两个文件的代码

__config.db.php__

{% highlight php %}

config.db.php
<?php
 
/*
 * All database connection variables
 */
 
define('DB_USERNAME', "root"); // User
define('DB_PASSWORD', ""); // Passwd
define('DB_DATABASE', "dbmeizi"); // Database
define('DB_HOST', "127.0.0.1"); // Server
define('DB_PORT', 3307); // Port
?>
{% endhighlight %}

__database.class.php__

{% highlight php %}
database.class.php
<?php
 
/**
 * A class file to connect to database
 */
class DbConnect {
 
    // constructor
    function __construct() {
    }
 
    // destructor
    function __destruct() {
    }
 
    /**
     * Function to connect with database
     */
    function connect() {
        // import database connection variables
        require_once __DIR__ . '/config.db.php';
 
        // Connecting to mysql database
        $this->conn = new mysqli(DB_HOST, DB_USERNAME, DB_PASSWORD, DB_DATABASE, DB_PORT);
 
        // Check for database connection error
        if (mysqli_connect_errno()) {
            echo "Failed to connect to MySQL: " . mysqli_connect_error();
        }
        $this->conn->query("SET NAMES utf8"); 
        // returing connection resource
        return $this->conn;
    }
}
 
?>
{% endhighlight %}

说明：在接下来的工程中，当我们需要使用数据库连接的时候，可以使用下面的代码：

	$db = new DbConnect(); 
	$conn = $db->connect();
	
	
## 通过PHP进行基本的MySQL CRUD操作

接下来进行通过PHP操作数据库，CRUD代表Create, Read, Update, Delete 

### 1.Reading All Rows from MySQL

要读取数据库中我们抓取的meizi的列表，创建一个文件 __girllist.php__ ，写入下面的代码
{% highligth php %}
<?php
 
/*
 * get girl list 
 */
 
$response = array();
 
require_once __DIR__ . '/include/database.class.php'; 
 
// connecting to db
$db = new DbConnect();
$conn = $db->connect();


// 默认页数
$size = 10;
$page = $_GET['p'];
if ($page == '' || !is_numeric($page)) {
    $page = 0;
}
$start = $size * intval($page);
 
// get girl list from table

// new girl
$sql = "SELECT * FROM meizi WHERE 1 = 1 ORDER BY id DESC LIMIT {$start}, {$size}";

$result = $conn->query($sql)->fetch_all(MYSQLI_ASSOC);
if (!empty($result)) {
    // check for empty result
    // success
    $response["code"] = 1;
    $response["msg"] = "success";
    $response["list"] = $result;
    echo json_encode($response);
} else {
    // no girl found
    $response["code"] = 0;
    $response["msg"] = "No girl found";
    $response["list"] = array();
    echo json_encode($response);

}
    
?>
{% endhighligth %}

在浏览器访问这个文件，成功获取列表的时候：
	
	{
		code: 1,
		msg: "success",
		list: [
			{
				id: "160",
				title: "睡不着 秒回么？",
				imgsrc: "http://ww2.sinaimg.cn/bmiddle/0060lm7Tgw1evush3zicwj30dw0iu41b.jpg",
				topic_link: "http://www.dbmeinv.com/dbgroup/417098",
				star_count: "0",
				update_time: "1441697041"
			},
			{
				id: "159",
				title: "【晒】",
				imgsrc: "http://ww2.sinaimg.cn/bmiddle/0060lm7Tgw1evusfx74ipj30dw0iitbl.jpg",
				topic_link: "http://www.dbmeinv.com/dbgroup/417122",
				star_count: "0",
				update_time: "1441697041"
			}
		]
	}

### 2.Updating a Row in MySQL

之前的系列中提到过，用户可以给某个meizi点赞，实现这个功能创建一个文件 __likegirl.php__ ，写入下面的代码

{% highligth php %}
<?php
 
/*
 * like a girl 
 */
 
// array for JSON response
$response = array();
 
// include db connect class
require_once __DIR__ . '/include/database.class.php'; 
 
// connecting to db
$db = new DbConnect();
$conn = $db->connect();
 

// id of the girl
$girlid = $_GET['girlid'];
if ($girlid == '' || !is_numeric($girlid)) {
    $response["code"] = 0;
    $response["msg"] = "oops, you need to tell me the girl's id";
     exit(json_encode($response));
}
// let the girl's startcount + 1
$result = $conn->query("UPDATE meizi set star_count = star_count + 1 WHERE id = {$girlid}");

if ($result) {
    // check for empty result
    // success
    $response["code"] = 1;
    $response["msg"] = "success";
    echo json_encode($response);
} else {
    // no girl found
    $response["code"] = 0;
    $response["msg"] = "failed";
    echo json_encode($response);
}

?>
{% endhighligth %}

在浏览器中访问这个文件，注意要拼接上`girlid`参数和对应的值。成功点赞的时候：

	{
		code: 1,
		msg: "success"
	}
	
至此，我们已经实现了跟客户端通信的一套基本框架，其他功能都可以在这个基础上拓展。

## 最后

按照惯例，放上源码地址：




