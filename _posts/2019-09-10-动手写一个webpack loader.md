---
layout:     post
title:      动手写一个webpack loader
subtitle:   loader
date:       2019-09-10
author:     HandsomeWalker
header-img: img/post-bg-js-version.jpg
catalog: true
tags:
    - Web前端
    - Javascript
---

## 简介
首先看一段webpack配置项
```javascript
{
  module: {
    rules: [
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: "url-loader",
        options: {
          limit: 10000,
          name: utils.assetsPath("imgs/[name].[hash:7].[ext]")
        }
      }
    ]
  }
}
```
没错，如果是用vue-cli 2.x搭建的项目，基本都能在build/webpack.base.conf.js里找到类似的。

那么什么是loader？一个单一功能的函数。有什么用？webpack官方文档说得很清楚

>loader 可以使你在 import 或"加载"模块时预处理文件

也就是说，**每当你在某个vue文件里import某个文件的时候，loader便会对该文件预先进行处理，然后在给到前端。**

回头在看上面的webpack配置，可以很明显的知道这个叫url-loader的loader，对图片后缀的文件进行处理。

## 动手写一个loader
那么为了进一步的了解其原理，可以试着动手写一个最简单的单一loader。

我们新建一个my-loader.js放在项目根目录

./my-loader.js
```javascript
const loaderUtils = require("loader-utils");
const validateOptions = require("schema-utils");
// 验证options
const schema = {
  type: 'object',
  properties: {
    firstWord: {
      type: 'boolean'
    }
  }
};

module.exports = function(source) {
  // 接收的options配置
  const options = loaderUtils.getOptions(this);
  validateOptions(schema, options, 'My Loader');

  res = source;
  // 如果firstWord为true，则只返回文本的第一个字
  if(options.firstWord) {
    res = res[0];
  }
  return `export default ${ JSON.stringify(res) }`;
}
```
然后修改webpack.base.conf.js配置，将my-loader加入到rules数组里

./build/webpack.base.conf.js
```javascript
{
  module: {
    rules: [
      {
        test: /\.txt$/,
        loader: resolve("my-loader.js"),
        options: {
          firstWord: true
        }
      }
    ]
  }
}
```
最后在项目根目录新建一个test.txt

./test.txt
```txt
hello world
```

接下来进行测试，在src/App.vue里面引入test.txt文件，然后打印结果

./App.vue
```javascript
import test from "../test.txt";
console.log(test);
```

OK，如果没有差错的话，浏览器打印一个 "**h**"，把firstWord去掉，结果是"**hello world**"。

至此已经完成一个简单的loader的自定义，其实自定义loader的使用主要看某些特殊的场景，一般情况下理解loader的用途即可。