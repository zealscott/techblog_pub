+++
title = "Nodejs 豆瓣爬虫实践"
slug = "nodejs crawler"
tags = ["nodejs"]
date = "2020-05-01T20:31:03+08:00"
description = "使用 Nodejs 从豆瓣小组中爬取帖子，并进行过滤"

+++

## 前端网页解析

### 网页结构

打开一个豆瓣小组网页，例如

```
https://www.douban.com/group/16473/
```

使用 F12 解析网站，可以看到，每一个帖子都由一个`a`标签构成，标题为`title`

![](/images/misc/douban1.png)

我们需要提取的包括`标题`、`URL`以及`时间`信息，因此可以直接使用`request`以及`cheerio`包进行提取：

```js
request(opt, function (error, response, body) {
      if (error) {
        return cb(error);
      }
      var $ = cheerio.load(body, {
        normalizeWhitespace: true,
        decodeEntities     : false
      });
      var items = [];
      $('.olt tr').slice(1).each(function (index, el) {
        var title = $('.title > a', el).attr('title');
        if (checkIncludeWords(title, includeWords) && checkExcludeWords(title, excludeWords)) {
          var item = {
            title   : title,
            href    : $('.title > a', el).attr('href'),
            lastTime: $('.time', el).text()
          };
          console.log('fetch item', item);
          items.push(item);
        }
      });
```

### 发送请求

注意，直接发送 request 请求，会出现 403 错误，因此我们需要伪造报文头部，这里伪造的是 Mac 电脑上的 Chrome（也就是我电脑上的配置）：

```js
/**
 * 请求参数初始化
 * @param: urls:请求的url、 opts：请求header参数， num：爬取的页数
 */
function initRequestOpt(urls, opts, num) {
  urls.forEach(function (url) {
    for (var i = 0; i < num; i++) {
      opts.push({
        method : 'GET',
        url    : url,
        qs     : {start: (i * 25).toString()},
        headers: {
          'Accept'       : '*/*',
          'Accept-Language': 'zh-CN,zh;q=0.8',
          'Cookie': 'bid=vkXjYPjxO6E; ll="108258";',
          'User-Agent'   : 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.95 Safari/537.36'
        }
        // 伪造报文头部，模仿浏览器行为，否则403错误
      });
    }
  });
}
```

## 关键词筛选

在进行网页爬虫时，我们经常会对网页进行筛选，例如查看租房信息，我们需要将「仅限女生」的信息排除；查看打羽毛球的咨询，需要将「双打」的去掉。同时，我们有时候又需要包含某些关键字，例如「约球」等等。因此，我们使用正则表达式来处理。

我们将需要排除的字符串及需要包含的字符串用数组作为参数传入。

```js
/**
 * 关键词筛选
 * @param: str:被筛选的字符串、 words：正则表达式参数（数组）
 * @return: true:包含所有关键词、 false:不全包含给出的关键词
 */
function checkIncludeWords(str, words) {
  var result = words.every(function (word) {
    return new RegExp(word, 'g').test(str);
  });
  return result;
}

/**
 * 关键词排除
 * @param: str:被筛选的字符串、 words：正则表达式参数（数组）
 * @return: true:不包含任一关键词、 false:包含给出的关键词
 */
function checkExcludeWords(str, words) {
  var result = words.some(function (word) {
    return new RegExp(word, 'g').test(str);
  });
  return !result;
}
```

## 主函数

### 入口

我们规定爬取的数量、关键字、初始页面，使用`eventproxy`这个包简化代码，并将结果保存在当前目录下的 json 文件：

```js
var request = require("request");
var cheerio = require("cheerio");
var eventproxy = require('eventproxy');
var ep = new eventproxy();
var fs = require('fs');

var pageNum = 10, requestOpts = [];
var topicUrls = [
  'https://www.douban.com/group/106955/discussion',
  'https://www.douban.com/group/nanshanzufang/discussion'
];

initRequestOpt(topicUrls, requestOpts, pageNum);

console.log('=======  start fetch  =======');
fetchData(
  ["约球"],
  ["仅限女生","仅限双打"],
  function (err, results) {
    if (err) {
      console.log('err', err);
    } else {
      fs.writeFile('output.json', JSON.stringify(results, null, 2),function(err, result) {
        if(err) console.log('error', err);
      });
    }
  }
);
```

### Request

```js
/**
 * 爬取数据主程序
 * @param:
 *    includeWords: <array>标题中需要包含的关键词数组
 *    excludeWords：<array>标题中需要排除的关键词数组
 */
function fetchData(includeWords, excludeWords, cb) {
  // 爬取各页数据
  ep.after('topic_titles', pageNum, function (topics) {
    // topics 是个数组，包含了 n次 ep.emit(event, cbData) 中的cbData所组成的数组
    // 由于在event中已经是数组，所以这里得到的是数组的数组，下面处理可以摊平它
    var results = topics.reduce(function (pre, next) {
      return pre.concat(next);
    });
    cb(null, results);
  });

  requestOpts.forEach(function (opt) {
    request(opt, function (error, response, body) {
      if (error) {
        return cb(error);
      }
      var $ = cheerio.load(body, {
        normalizeWhitespace: true,
        decodeEntities     : false
      });
      var items = [];
      $('.olt tr').slice(1).each(function (index, el) {
        var title = $('.title > a', el).attr('title');
        if (checkIncludeWords(title, includeWords) && checkExcludeWords(title, excludeWords)) {
          var item = {
            title   : title,
            href    : $('.title > a', el).attr('href'),
            lastTime: $('.time', el).text()
          };
          console.log('fetch item', item);
          items.push(item);
        }
      });
      // 发布单个请求完成事件并返回结果（数组）
      ep.emit('topic_titles', items);
    });
  });

}
```

## 结果展示

使用「租房」作为关键字，并将「仅限女生」的结果为去掉：

![hugo-logo-wide](/images/misc/douban2.png)

使用「约球」作为关键字，只显示在「上海」的约球，并将「仅限单打」的去掉：

![hugo-logo-wide](/images/misc/douban3.png)

保存为 json 文件为：

```json
[
  {
    "title": "上海金山石化招募球友",
    "href": "https://www.douban.com/group/topic/71605266/",
    "lastTime": "2019-10-04"
  },
  {
    "title": "【➤上海羽毛球微信群，欢迎新手加入我们】",
    "href": "https://www.douban.com/group/topic/172391054/",
    "lastTime": "05-07 09:44"
  },
  {
    "title": "上海 💪🏻运动 约伴 交友群🥘👫",
    "href": "https://www.douban.com/group/topic/158495696/",
    "lastTime": "05-06 13:56"
  },
  {
    "title": "上海交通大学附近求球友",
    "href": "https://www.douban.com/group/topic/172797653/",
    "lastTime": "05-05 14:48"
  },
  {
    "title": "上海闵行梅陇附近打羽毛球",
    "href": "https://www.douban.com/group/topic/174202556/",
    "lastTime": "05-05 10:03"
  }
]
```
