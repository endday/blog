---
title: cookies的基本操作方法
url: https://www.yuque.com/endday/blog/unuacy
---

<a name="119935d0"></a>

## cookie 的属性选项

| 属性名 | 描述 |
| :---: | --- |
| name(string) | 该Cookie的名称。Cookie一旦创建，名称便不可更改 |
| value(string) | 该Cookie的值。如果值为Unicode字符，需要为字符编码。如果值为二进制数据，则需要使用BASE64编码 |
| maxAge(number) | 该Cookie失效的时间，单位秒。如果为正数，则该Cookie在maxAge秒之后失效。如果为负数，该Cookie为临时Cookie，关闭浏览器即失效，浏览器也不会以任何形式保存该Cookie。如果为0，表示删除该Cookie。默认为–1 |
| secure(booblean) | 该Cookie是否仅被使用安全协议传输。安全协议。安全协议有HTTPS，SSL等，在网络上传输数据之前先将数据加密。默认为false |
| path(string) | 该Cookie的使用路径。如果设置为“/sessionWeb/”，则只有contextPath为“/sessionWeb”的程序可以访问该Cookie。如果设置为“/”，则本域名下contextPath都可以访问该Cookie。注意最后一个字符必须为“/” |
| domain(string) | 可以访问该Cookie的域名。如果设置为“.google.com”，则所有以“google.com”结尾的域名都可以访问该Cookie。注意第一个字符必须为“.” |
| comment | 该Cookie的用处说明。浏览器显示Cookie信息的时候显示该说明 |
| version(number) | 该Cookie使用的版本号。0表示遵循Netscape的Cookie规范，1表示遵循W3C的RFC 2109规范 |

<a name="8f40db3b"></a>

## 赋值

document.cookie看上去就像一个属性，可以赋不同的值。但它和一般的属性不一样，改变

它的赋值并不意味着丢失原来的值，例如连续执行下面两条语句：

    document.cookie="userId=828"
    document.cookie="userName=hulk"

这时浏览器将维护两个cookie，分别是userId和userName，因此给document.cookie赋值更像执行类似这样的语句：

    document.addCookie("userId=828")
    document.addCookie("userName=hulk")

事实上，浏览器就是按照这样的方式来设置cookie的，如果要改变一个cookie的值，只需重新赋值，例如：

    document.cookie="userId=929"

这样就将名为userId的cookie值设置为了929。

<a name="adf91a3e"></a>

## 取值

下面介绍如何获取cookie的值。cookie的值可以由document.cookie直接获得：

var strCookie=document.cookie;

这将获得以分号隔开的多个名/值对所组成的字符串，这些名/值对包括了该域名下的所有cookie。

例如：

    document.cookie="userId=828";
    document.cookie="userName=hulk";
    var strCookie=document.cookie;
    alert(strCookie);

只能够一次获取所有的cookie值，而不能指定cookie名称来获得指定的值。

获取指定的cookie值，例如，要获取userId的值，可以这样实现：

    function getCookies(cookiesName) {
        const strCookie = document.cookie
        const arrCookie = strCookie.split('; ')
        let value
        for (let i = 0; i < arrCookie.length; i++) {
            const arr = arrCookie[i].split('=')
            if (arr[0] === cookiesName) {
                value = arr[1]
                break
            }
        }
        return value
    }
    const userId = getCookies('userId')

<a name="fe8e9c0a"></a>

## 设置终止日期

<a name="bc4014a3"></a>

## 删除cookie

<a name="70e99862"></a>

## 指定可访问cookie的路径
