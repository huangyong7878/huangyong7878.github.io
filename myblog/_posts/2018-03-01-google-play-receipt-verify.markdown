---
layout: post
title:  "验证in-app-purchase receipt!"
date:   2018-03-01 18:36:30 +0800
categories: Android
---
# 验证收据步骤
##. 创建OAuth Cred
1. 访问 [api console][google-api]
2. 左上角选择project
3. 左上角选择 create credentials -> OAuth ->Web Application
4. redirect URL 填入 https://developers.google.com/oauthplayground
5. 记下 clientID,clientSecret
6. 现在你需要refreshtoken，访问 https://developers.google.com/oauthplayground
7. 右上角选择Gear setting，选择 'User your own OAuth credentials'
8. 左边选择 ‘Google Play Developer API v2’
8. 点击 'Authorization  Api'
9. Save Authorization code
10. 点击 'Exchange Authorization code  for token'
11. Save Refresh token

## 如何认证

```javascript 
function auth(){
var body = {};
	body[KEYS.GRANT_TYPE] = KEYS.REFRESH_TOKEN;
	body[KEYS.CLIENT_ID] = tokenMap.clientId;
	body[KEYS.CLIENT_SECRET] = tokenMap.clientSecret;
	body[KEYS.REFRESH_TOKEN] = tokenMap.refreshToken;

	var options = {
		method: 'POST',
		url: 'https://accounts.google.com/o/oauth2/token',
		form: body,
		json: true
	};

	request(options, cb);
}


auth(tokenMap, function (error, body) {
		if (error) {
			return cb(error, {
				status: constants.VALIDATION.FAILURE,
				message: error.message
			});
		}

		verbose.log(NAME, 'Google API authenticated', body.body);

		// well....
		var accessToken = body.access_token || body.body.access_token;

		verbose.log(NAME, 'Google API access token:', accessToken);

		if (!accessToken) {
			return cb(new Error(JSON.stringify(body)), {
				status: constants.VALIDATION.FAILURE,
				message: 'failed to authenticate api call'
			});
		}
		var url = 'https://www.googleapis.com/androidpublisher/v2/applications/' +
			encodeURIComponent(receipt.data.packageName) +
			'/purchases/products/' +
			encodeURIComponent(receipt.data.productId) +
			'/tokens/' + encodeURIComponent(receipt.data.purchaseToken) +
			'?access_token=' + encodeURIComponent(accessToken);
		request.get({
			url: url,
			json: true
		}, _onProductValidate);
	});

```


[google-api]: https://console.developers.google.com/apis/credentials
