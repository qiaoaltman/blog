---
layout: post
title: "facebook api login share and instagram login"
date: 2018-04-23 19:37:27 +0800
comments: true
categories: facebook
---
本篇文章主要介绍三方登录 facebook instagram

分为四个部分

1 .facebook api login

2 .facebook user info

3 .instagram api login

<!--more-->

### facebook api login

首先注册成为开发者然后注册自己的APP 选择网站website https://developers.facebook.com

url redirecturl 都设置为web app的域名

关于隐私链接 [隐私链接](https://ancientbook.cn/fbprivacy.html)

然后把代码放入页面
```
<fb:login-button
        scope="publish_actions"
        onlogin="checkLoginState();">
</fb:login-button>
<script>
window.fbAsyncInit = function () {
    FB.init({
        appId: '386610931818049', // appId
        cookie: true,
        xfbml: true,
        version: 'v2.12'    // api version
    });

    FB.AppEvents.logPageView();

};
// load facebook sdk
(function (d, s, id) {
    var js, fjs = d.getElementsByTagName(s)[0];
    if (d.getElementById(id)) {
        return;
    }
    js = d.createElement(s);
    js.id = id;
    js.src = "https://connect.facebook.net/en_US/sdk.js";
    fjs.parentNode.insertBefore(js, fjs);
}(document, 'script', 'facebook-jssdk'));

// login callback
function checkLoginState() {
    FB.getLoginStatus(function (response) {
        statusChangeCallback(response);
    });
}

// callback
function statusChangeCallback(response) {
    // get user picture through api
    FB.api(
        '/' + response.authResponse.userID + '/picture',
        'GET',
        {'redirect': '0'},
        function (response) {
            // Insert your code here
            console.log(response);
        }
    );
    console.log(JSON.stringify(response))
}
</script>

```



参考链接 https://developers.google.com/identity/sign-in/web/sign-in