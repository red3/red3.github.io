---
layout: post
title:  "Scrapy+MySQL+PHP+Swift开发攻略系列（六）之RESTAPI篇"
date:   2015-09-10 14:30:00
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

## 温故而知新

上个系列中，我们编写了客户端跟服务器数据库沟通的接口文件。

考虑到我们将会编写iOS客户端程序，或许将来还会有Android程序和Web App，那么构建`REST API`来进行App和服务器的沟通将是一个很好的选择。

本次系列将会构建一个简单的`REST API`.

## 准备工作

1. 关于`REST API`，参考这个[链接](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)了解
2. HTTP Methods：设计良好的 RESTful API 应该支持主流的请求方法，并且依据具体的操作类型在选择使用哪中方法的时候应该取决于操作类型

	- __GET__ ： 拉取一条资源
	- __POST__ ：新建一条资源
	- __PUT__ ：更新一条资源 
	- __DELETE__ ：删除一条资源

3. Slim PHP Micro Framework

	为了站在巨人的肩膀上，我们选择利用`Slim`这个框架来进行开发。 `Slim`作为一个支持REST API的开发框架，轻量且容易上手，可以从这个[链接](https://github.com/codeguy/Slim)下载。

4. 安装谷歌浏览器插件`Advanced REST client`来进行测试

在Chrome中通过这个[链接](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo)安装`Advanced REST client`插件，因为众所周知的原因，需要自备梯子。

## 开始

基于之前系列的基础，我们把API的目录架构设计下面的样子：

	RestfulApi
		|__ include
		 		|__ config.db.php
		 		|__ database.class.php
		|__	 libs
		 		|__ slim/
		|__ v1
				|__ index.php
		
- libs – 放置第三方框架
- include – 放置数据库链接及配置等文件
- index.php – 处理接口请求的文件

在 __v1__ 文件夹中创建 __index.php__ 文件，加入下面的代码。这里我们主要加入了需要引用的类库以及一些函数。

__echoRespnse()__ – 输出json格式的返回值以及请求状态码。

``` php
<?php

require '../libs/Slim/Slim.php';
require_once '../include/database.class.php'; 

\Slim\Slim::registerAutoloader();
$app = new \Slim\Slim();

// connecting to db
$db = new DbConnect();
$conn = $db->connect();

/**
 * Echoing json response to client
 * @param String $status_code Http response code
 * @param Int $response Json response
 */
function echoRespnse($status_code, $response) {
    $app = \Slim\Slim::getInstance();
    // Http response code
    $app->status($status_code);
 
    // setting response content type to json
    $app->contentType('application/json');
 
    echo json_encode($response);
}
$app->run();
```

### => Getting All Girls

下面的代码会拉取服务器上的一组列表下来，考虑到数据比较多，可以利用`p`和`size`参数来约束要拉取的数据。

- size：每次拉取的数据的数量
- p：分页，假设size=10，p=1就会拉取数据库中前10条数据，p=2就会拉取服务器第10-20条数据

``` php
$app->get('/girls', function () use ($app, $conn) {
    $page = $app->request->get('p');
    if ($page == '' || !is_numeric($page)) {
   		$page = 0;
   	}
	$size = $app->request->get('size');
   	// let the default size be 10
   	if ($size == '' || !is_numeric($size)) {
   		$size = 10;
   	} 
   	$start = $size * intval($page);
 
	// get girl list from table
   	if ($_GET['ishot']) {
    // hot girl
   		$sql = "SELECT * FROM meizi WHERE 1 = 1 ORDER BY star_count DESC, id DESC LIMIT {$start}, {$size} ";
   	} else {
    // new girl
   		$sql = "SELECT * FROM meizi WHERE 1 = 1 ORDER BY id DESC LIMIT {$start}, {$size}";
   	}

   	$result = $conn->query($sql)->fetch_all(MYSQLI_ASSOC);
   	if (!empty($result)) {

    // check for empty result
    // success
   		$response["code"] = 1;
   		$response["msg"] = "success";
   		$response["list"] = $result;
   		echoRespnse(200, $response);
   	} else {
    // no girl found
   		$response["code"] = 0;
   		$response["msg"] = "No girl found";
   		$response["list"] = array();
   		echoRespnse(200, $response);

   	}
});

```

- __URL__ : /girls
- __Method__ : __GET__
- __Params__ : p, size

请求成功发出后，下面的json将会输出：

	
	{
	code: 1
	msg: "success"
	list: [2]
		0:  {
			id: "160"
			title: "睡不着 秒回么？"
			imgsrc: "http://ww2.sinaimg.cn/bmiddle/0060lm7Tgw1evush3zicwj30dw0iu41b.jpg"
			topic_link: "http://www.dbmeinv.com/dbgroup/417098"
			star_count: "0"
			update_time: "1441697041"
			}
		1:  {
			id: "159"
			title: "【晒】"
			imgsrc: "http://ww2.sinaimg.cn/bmiddle/0060lm7Tgw1evusfx74ipj30dw0iitbl.jpg"
			topic_link: "http://www.dbmeinv.com/dbgroup/417122"
			star_count: "0"
			update_time: "1441697041"
			}
	}

### => Updating a girl's star

下面的代码将会更新某一个girl的`star_count`，为了定位到具体的girl，采用url结构 __girls/:girlid__ 的形式，具体请求的时候，需要用具体的id替换掉 __:girlid__.

``` php
$app->put('/girls/:girlid', function ($girlid) use ($app, $conn) {
	if ($girlid == '' || !is_numeric($girlid)) {
    	$response["code"] = 0;
    	$response["msg"] = "oops, you need to tell me the girl's id";
     	exit(json_encode($response));
	}

	// let the girl's star_count + 1
	$result = $conn->query("UPDATE meizi set star_count = star_count + 1 WHERE id = {$girlid}");

	if ($result) {
    	// success
		$response["code"] = 1;
		$response["msg"] = "success";
		echoRespnse(200, $response);
	} else {
    	// no girl found
		$response["code"] = 0;
		$response["msg"] = "failed";
		echoRespnse(200, $response);
	}
});

```

-  URL : /girls/girlid (girlid要用girlid的字段值替换)
- __Method__ : __PUT__ 
- __Params__ : -

成功调用接口，会看到如下的打印：

	{
		code: 1
		msg: "success"
	}

## 测试接口

## 最后

按照惯例，放上源码地址：




