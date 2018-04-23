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
<!-- 这里放到div 当然如果不想使用默认button 还可以自定义-->
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

登录成功后返回的用户信息为
```json
{
"authResponse":{
"accessToken":"EAAFfnt9B3kEBAMsIzz4PZBYpDivuopRYzrOGew8oMZC55PaDJZByHEid2Cpm9aUf5R6qroHZCsJ3Aa7ENC44I8LZC2gjcOC6wnovGWjDhXZAkAatXKZCVkeUHFINUc5qZAv3pRN7ZAlnor8SNVtmZBqWWcnYuEMZCIC87EC0Ov4SisDGDvbgQ2kHCdtDlZBUZC1CX5uBFC7WnZBMDAhJDZA00nhEA0g",
"userID":"110712723123784",
"expiresIn":5101,
"signedRequest":"TOEoGHJcXPZTQgqVeaynfYJRGi0EnCbXl2uxILk7_mE.eyJhbGdvcml0aG0iOiJITUFDLVNIQTI1NiIsImNvZGUiOiJBUUJjUVkxQWZRV2lJeHc5SHdYOW5SYXN0Y2xVX3RTVE1rQlVRU0hvTXBFbTVVaWRyYWdwcTdReXhCMTh0djhlTC0yeTQ2UjNDS3ZQV1lrV1dSTldUVzVNRmlWNlZ5X0pObVhZeVBmMDd1MlEyVzRnSHZBejhWdDkyMEN0NHFOVWg5YTJNNDh5cmItb09IU3c0VmxiWUJyZ1RGamFISlVKUVF3Um9KRlYzZVZkdXF1anpuNkM4WnpDOGRUZTJjVXc1NllBSzhzQjNqY21JTkNNUFllWGVwV1lPR0xQYVdZYWtQSW9nS2NEVS1teFNrOV9RZmU2cGJoRGZCbkNvQVVfa0E1QmY0MnBKT1g5RFBaUzBBU3doZU9UaTdsVGNycUZFRkt6c3F6VTVkQTFmOVk1c211NHFQdU50S3lFaEtsMzV5eUMyV0NvMTRrTnhWNEZCTFBYQVFwdCIsImlzc3VlZF9hdCI6MTUyNDQ1ODA5OSwidXNlcl9pZCI6IjExMDcxMjcyMzEyMzc4NCJ9"
},
"status":"connected"
}
```

自定义按钮
如果應用程式想要使用自己的按鈕，只需呼叫 FB.login()，即可叫用「登入」對話方塊：

```
FB.login(function(response){
  // Handle the response object, like in statusChangeCallback() in our demo
  // code.
});
```

根据accessToken 和 userID可以获取更详细的用户信息

```
https://graph.facebook.com/v2.12/110712723123784/picture?access_token=EAAFfnt9B3kEBAOb7ZCYuO3f4LSeQ3nZCkKKiV1za55ZBFSxhemumm35u0cAK9ZBNXb2bhe4venF5t8UcpjyKaD7xjeMas56QZAsxVL1ySb2JA9e5u81kHYtrTHZCU8EqMdLRPcrs4duVK8ie0B0ZCBTe9zg8b1Ba4GtyX9f0ZC83OdRqEjXsiLQi2ehaZCeFTXDYrK0ILnbZB7aZB2EYCZAkZAB9X&method=get&pretty=0&redirect=0&sdk=joey&suppress_http_code=1
```

返回信息
```json
{"data":{"height":50,"is_silhouette":false,"url":"https:\/\/lookaside.facebook.com\/platform\/profilepic\/?asid=110712723123784&height=50&width=50&ext=1524731636&hash=AeQjW1S7SKvTa3Sy","width":50}}
```

这里有一个问题 返回的图片链接打开直接就被下载了

解决办法参考这个 issue https://github.com/bumptech/glide/issues/2986

### Instagram api login

也是需要有自己的client 获取到clientID和clientSecret  https://www.instagram.com/developer/authentication/

请求这个链接 https://api.instagram.com/oauth/authorize/?client_id=CLIENT-ID&redirect_uri=REDIRECT-URI&response_type=code

把clientID redirect_uri改为自己的

就会得到http://redirect_uri?code=xxx

然后 用服务端发送请求获取用户信息

```
curl -F 'client_id=CLIENT_ID' \
    -F 'client_secret=CLIENT_SECRET' \
    -F 'grant_type=authorization_code' \
    -F 'redirect_uri=AUTHORIZATION_REDIRECT_URI' \
    -F 'code=CODE' \
    https://api.instagram.com/oauth/access_token
```

-F 表示FORM DATA

然后得到返回信息

```json
{"access_token": "7574672867.29ceb57.e7ee82776c934c7492ecc0a73a08dd49", "user": {"id": "7574672867", "username": "tangxuelong", "profile_picture": "https://scontent.cdninstagram.com/vp/f17e1275a70fc32e93cbf434ddc32bcd/5B6CCC7A/t51.2885-19/11906329_960233084022564_1448528159_a.jpg", "full_name": "\u5510\u96ea\u9f99", "bio": "", "website": "", "is_business": false}}
```

### Google api login

参考链接 https://developers.google.com/identity/sign-in/web/sign-in