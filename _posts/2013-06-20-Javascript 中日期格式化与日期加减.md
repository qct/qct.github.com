---
layout: post
title: "Javascript 中日期格式化与日期加减"
description: "网上看到很多的javascript日期加减都是 

    var myDate=new Date()
    myDate.setDate(myDate.getDate()+5)
    
这种想在当前日期上加5天的做法碰到月末就有bug，比如6月28号 加 5天 是 6月33号。 自己想了下，还是通过 `getTime()` 方法拿到日期锁代表的毫秒数，然后在毫秒数上做加减，最后返回一个新的`Date`对象，也省去了对其它 年、月、日 等字段进行判断、运算了。比如：
"
category: 技术
tags: [javascript]
---
{% include JB/setup %}

### Javascript 日期格式化

一个很方便的函数，把javascript中`Date`对象 格式化成 字符串，类似java中的`SimpleDateFormat`：

    Date.prototype.format = function(format){
     /*
      * eg:format="yyyy-MM-dd hh:mm:ss";
      */
     var o = {
      "M+" :  this.getMonth()+1,  //month
      "d+" :  this.getDate(),     //day
      "h+" :  this.getHours(),    //hour
      "m+" :  this.getMinutes(),  //minute
      "s+" :  this.getSeconds(), //second
      "q+" :  Math.floor((this.getMonth()+3)/3),  //quarter
      "S"  :  this.getMilliseconds() //millisecond
    }
     
       if(/(y+)/.test(format)) {
        format = format.replace(RegExp.$1, (this.getFullYear()+"").substr(4 - RegExp.$1.length));
       }
     
       for(var k in o) {
        if(new RegExp("("+ k +")").test(format)) {
          format = format.replace(RegExp.$1, RegExp.$1.length==1 ? o[k] : ("00"+ o[k]).substr((""+ o[k]).length));
        }
       }
     return format;
    }
    
    
例如： `new Date().format('yyyy-MM-dd hh:mm:ss')` 对应格式化后的字符串： 

> 2013-06-20 14:43:20


### 日期加减

网上看到很多的javascript日期加减都是 

    var myDate=new Date()
    myDate.setDate(myDate.getDate()+5)
    
这种想在当前日期上加5天的做法碰到月末就有bug，比如6月28号 加 5天 是 6月33号。 自己想了下，还是通过 `getTime()` 方法拿到日期锁代表的毫秒数，然后在毫秒数上做加减，最后返回一个新的`Date`对象，也省去了对其它 年、月、日 等字段进行判断、运算了。比如：

    Date.prototype.addDay = function(days) {
      return new Date(this.getTime() + 24 * 60 * 60 * 1000*days);
    }
    
如果当前日期为6月1日，那么 `new Date().addDay(-1)` 返回的是5月31。


如果想对其它字段进行加减，方法也类似，比如月份 年份 小时 分钟 等等。
