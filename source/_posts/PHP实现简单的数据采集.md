---
title: PHP实现简单的数据采集
abbrlink: ca3fda64
date: 2019-12-15 03:05:09
tags:
	- PHP
	- 数据采集
---

# 引言

​		说到数据采集大家首先都会想到python,代码简洁，高效，很容易就可以实现数据采集。

​		那PHP如何实现数据采集呢？非常简单。

# 概念

​		那什么是数据采集呢？以下是百度百科的介绍：

​		**数据采集，又称[数据获取](https://baike.baidu.com/item/数据获取)，是利用一种装置，从系统外部采集数据并输入到系统内部的一个接口。数据采集技术广泛应用在各个领域。**

> 你可以简单的理解为偷别人网站的数据。

# 需要的扩展包

## 1. [Guzzle](http://docs.guzzlephp.org/en/stable/index.html)

> 这是一个PHP HTTP客户端，可以轻松发送HTTP请求并轻松与Web服务集成。

安装方式：

```php
composer require guzzlehttp/guzzle:~6.0
```

或者：

在composer.json加入

```json
"require": {
      "guzzlehttp/guzzle": "~6.0"
   }
}
```

## 2. [QueryList](https://querylist.cc/docs/guide/v3/overview)

​		QueryList是一个基于phpQuery的PHP通用列表采集类,得益于phpQuery，让使用QueryList几乎没有任何学习成本，只要会CSS3选择器就可以轻松使用QueryList了，它让PHP做采集像jQuery选择元素一样简单。 QueryList的几个特点:

1. 学习简单：只有一个核心的API
2. 使用简单：用jQuery选择器来选择页面元素
3. 自带过滤功能，可过滤掉无用的内容
4. 支持无限层级嵌套采集
5. 采集结果直接以采集规则以列表的形式有序的返回
6. 支持扩展

> 我们可以使用它来过滤html内容

安装方式：

```php
composer require jaeger/querylist:V3.2.1
```

# 采集案例

> 我们以 **LearnKu** 社区为例，我们将采集社区的帖子信息，并把这些信息存入文件和存入mysql数据库。

## 1.安装依赖

在命令行输入以下命令

```php
composer init
```

引入依赖

```json
{
    "require": {
        "guzzlehttp/guzzle": "~6.0@dev",     
        "jaeger/querylist": "V3.2.1"
    },
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        }
    }
}

```

安装依赖

```
composer install
```

## 2.采集类

app\Handle\ClientHandle.php

```php
<?php


namespace App\Handle;


use GuzzleHttp\Client;
use QL\QueryList;

class ClientHandle
{
    private $client;

    public function __construct()
    {
        $this->client = new Client(['verify' => false]);
    }

    public function queryBody($url, $rules)
    {
        $html = $this->sendRequest($url);

        $data = QueryList::Query($html, $rules)->getData(function ($item) {
            if (array_key_exists('link',$item)){
                $content = $this->sendRequest($item['link']);
                $item['post'] = QueryList::Query($content, [
                    'title' => ['div.pull-left>span', 'text'],
                    'review' => ['p>span.text-mute:eq(0)', 'text'],
                    'comment' => ['p>span.text-mute:eq(1)', 'text'],
                    'content' => ['div.content-body', 'html'],
                    'created_at' => ['p>a>span', 'title'],
                    'updated_at' => ['p>a:eq(2)', 'data-tooltip']
                ])->data[0];
            }
            return $item;
        });
//查看采集结果
        return $data;
    }

    private function sendRequest($url)
    {

        $response = $this->client->request('GET', $url, [
            'headers' => [
                'User-Agent' => 'testing/1.0',
                'Accept' => 'application/json',
                'X-Foo' => ['Bar', 'Baz']
            ],
            'form_params' => [
                'foo' => 'bar',
                'baz' => ['hi', 'there!']
            ],
            'timeout' => 3.14,
        ]);

        $body = $response->getBody();

//获取到页面源码
        $html = (string)$body;

        return $html;
    }
}
```



简单分析：

1. __construct 构造函数中我们实例化了一个 guzzleClient，用来发起http请求的。

2. sendRequest 是传入url，然后发起一个http请求并返回目标的html源码。

3. queryBody,接收一个url，和需要采集的规则，这里不做延伸 [queryList](https://querylist.cc/docs/guide/v3/start)，只要会使用jquery，那相信你很快上手。

   ```php
       public function queryBody($url, $rules)
       {
         //发起一个请求，接收html源码
           $html = $this->sendRequest($url);
         //将内容$html，和规则$rules 传给QueryList的静态方法Query处理，并获取数据。
           $data = QueryList::Query($html, $rules)->getData(function ($item) {
             //我首先获取的是列表页，然后通过列表的link链接再去获取文章的详细信息。
             
             //判断是否匹配到link
               if (array_key_exists('link',$item)){
                 //获取详情页的html源码
                   $content = $this->sendRequest($item['link']);
                 //再交给QueryList 处理数据
                   $item['post'] = QueryList::Query($content, [
                       'title' => ['div.pull-left>span', 'text'],
                       'review' => ['p>span.text-mute:eq(0)', 'text'],
                       'comment' => ['p>span.text-mute:eq(1)', 'text'],
                       'content' => ['div.content-body', 'html'],
                       'created_at' => ['p>a>span', 'title'],
                       'updated_at' => ['p>a:eq(2)', 'data-tooltip']
                   ])->data[0];
                 //采集到的是一个集合，所以我只取第一个 data[0]
               }
               return $item;
           });
   //查看采集结果
           return $data;
       }
   ```

## 3. PDO类

App\Handle\PdoHandle.php

> 我们使用PDO来操作数据库,这里我简单实现一个类

```php
<?php


namespace App\Handle;

class PdoHandle
{
    public $source;

    private $driver;
    private $host;
    private $dbname;
    private $username;
    private $password;

    /**
     * PdoHandle constructor.
     */
    public function __construct($driver = 'mysql', $host = 'localhost', $dbname = 'caiji', $username = 'root', $password = '')
    {
        $this->driver = $driver;
        $this->host = $host;
        $this->dbname = $dbname;
        $this->username = $username;
        $this->password = $password;
        $dsn = $this->driver . ':host=' . $this->host . ';dbname=' . $this->dbname;
        $this->source = new \PDO($dsn, $this->username, $this->password);
        $this->source->setAttribute(\PDO::ATTR_ERRMODE, \PDO::ERRMODE_EXCEPTION);
    }
}
```

相信都看得懂，就不介绍了

## 4. 写入文件

我们把采集到的内容，写入到文件里

```php
<?php
//设置请求时间无限制
set_time_limit(0);
//引入自动加载
require '../vendor/autoload.php';
//规则， 只取下标小于5的  也就是前5条数据
$rules = [
    'title' => ['span.topic-title:lt(5)', 'text'],
    'link' => ['a.topic-title-wrap:lt(5)', 'href']
];

//采集
$url = "https://learnku.com/laravel";
$client = new \App\Handle\ClientHandle();
$data = $client->queryBody($url, $rules);
//因为我们请求了两级，所以返回的数组需要处理成一级数组
$data = array_map(function ($item) {
    return $item['post'];
}, $data);

//写入文件
$handle = fopen('2.php','w');
$str = "<?php\n".var_export($data, true).";";
fwrite($handle,$str);
fclose($handle);
```

> 稍等几秒后，你就可以看到文件目录下多出个 2.php 的文件了，里面有数据代表采集成功~

## 5. 写入数据库

把采集到的内容写入到数据库里

### 1. 创建表

首先我们创建一张 posts表并有以下字段：

```
`title`, `review`, `comment`, `content`,`created_at`,`updated_at`
```

> created_at  和 updated_at 建议不要强制为时间类型和必填，否则需要再处理以下数据

### 2.操作

```php
<?php
set_time_limit(0);
require '../vendor/autoload.php';
$rules = [
    'title' => ['span.topic-title', 'text'],
    'link' => ['a.topic-title-wrap', 'href']
];
$url = "https://learnku.com/laravel";

$client = new \App\Handle\ClientHandle();
$data = $client->queryBody($url, $rules);
$data = array_map(function ($item) {
    return $item['post'];
}, $data);

//编写sql语句
$sql = "INSERT INTO `posts`(`title`, `review`, `comment`, `content`,`created_at`,`updated_at`) VALUES";

//再过滤下没有匹配到符合条件的特殊数据，避免入库的时候麻烦
$data = array_filter($data,function($item){
    return count($item) == 6;
});
//重置数组下标
sort($data);
//组合sql语句
foreach ($data as $key => $item) {
  //内容是有html标签的，所以我们要用 base64 处理下才能入库
    $item['content'] = base64_encode($item['content']);
    $value = "'" . implode("','", array_values($item)) . "'";
    $sql .= "($value)";
    if (count($data) - 1 != $key) {
        $sql .= ",";
    }
}

//采集
$db = new \App\Handle\PdoHandle();

try {
    $db->source->query($sql);
    echo '采集入库成功!';
} catch (PDOException $exception) {
    echo $exception->getMessage();
}
```

> 骚等几秒钟后，你就可以看到网页上输出 ‘采集入库成功’ 的字样，那代表成功了~

我们也可以只采集前几条，只需要重写**$rules**规则就行了

例如：只取前5条,我们可以这样写。

```php
$rules = [
    'title' => ['span.topic-title:lt(5)', 'text'],
    'link' => ['a.topic-title-wrap:lt(5)', 'href']
];
```

## 6. 读取数据

> 利用PDO读取数据

```php
<?php
require '../vendor/autoload.php';

$db = new \App\Handle\PdoHandle();
//查询
$sql = "select * from `posts` limit 0,10";

$pdoStatement = $db->source->query($sql);
$data = $pdoStatement->fetchAll(PDO::FETCH_ASSOC);

foreach ($data as &$item){
  //给内容解密
    $item['content'] = base64_decode($item['content']);
}

var_dump($data);
```

# 篇尾

希望对在看的你有点收获吧，同时我也把它上传到了github，需要的伙伴可以拉下来看下。

[案例](https://github.com/NaturalGao/guzzle-caiji)

[个人博客](https://naturalgao.github.io)