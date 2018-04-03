Cauly Hybrid App 연동 가이드
=========================
Android Native APP
--------------------------
### 개요
네이티브 앱은 Java 로 작성된 일반적인 Android 앱을 지칭합니다. 본 문서는 광고주의 앱이 네이티브 앱일 경우 하이브리드 연동을 하는 방법에 대해서 설명하는 문서입니다. 컨텐츠 제공에 Webview 를 주로 사용한다면 [Android Hybrid APP](https://github.com/CaulyTracker/Android-Tracking-SDK-Hybrid) 문서를 참고해주세요. 

### 문서 버전 
| 문서 버전 | 작성 날짜 | 작성자 및 내용 | 
| ---------- | ----------- | ---------------- |
| 1.0.0 | 2018.04.03 | 윤창주(yoonc1@fsn.co.kr) - 초안작성 |


### 목차
- [연동 절차](#연동-절차)
- [연동 상세](#연동-상세)
  - [Reference](#reference)

--------------------------

### 연동 절차
 
1. 코드 삽입 이후 연동이 되었는지 Cauly에게 APK 전달하여 테스트를 요청합니다.
1. Cauly에서 테스트를 완료하면 APP을 마켓에 업데이트하고 업데이트 내용을 Cauly와 공유합니다.
### 연동 상세
------------
#### Native APP SDK 연동
대상 OS 버전: Android 2.3 이상


##### Proguard
Proguard 적용시에는 SDK에 적용되지 않도록 아래 설정을 추가
```
-keep class com.fsn.cauly.tracker.** { *; }
```


#### Reference

----------
##### Webview를 사용하는 Hybrid App 적용 가이드
CaulyTracker Web SDK ( javascript version ) 을 사용는 Hybrid의 앱의 경우 App/Web의 더욱 정교한 Tracking 기능을 사용하고자 할 경우에는 [<i class="icon-file"></i> Cauly JS Inteface For WebView](#CaulyJSIntefaceForWebView) section을 참조해주세요.
 
###### samlple
```java
Webview web = new WebView(getApplicationContext());
web.addJavascriptInterface(new CaulyJsInterface(web),CaulyJsInterface.CAULY_JS_INTERFACE_NAME);
```

UIWebView를 사용하는 Hybrid App이 아닌 일반 브라우저에서 접근가능한 Web의 경우에는 해당 메시지를 호출하지않도록 조치를 해주어야 합니다.

--------------

Cauly JS Inteface For WebView
-----------------------------
##### Inject javascript interface
###### sample

```java
private WebView testWebview;

private final String TAG = "WEB";

@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_webview);

	testWebview = (WebView) findViewById(R.id.testWebview);

	testWebview.getSettings().setJavaScriptEnabled(true);
	testWebview.setWebChromeClient(new WebChromeClient() {
		@Override
		public boolean onJsAlert(final WebView view, final String url, final String message, JsResult result) {
			Log.d(TAG, "onJsAlert(!" + view + ", " + url + ", " + message + ", " + result + ")");
			Toast.makeText(getApplicationContext(), message, 3000).show();
			result.confirm();
			return true;
		}
	});
	testWebview.setWebViewClient(new WebViewClient());
	testWebview.clearCache(true);
	testWebview.loadUrl("http://[TESTURL]/test.html?t="+System.currentTimeMillis());
	testWebview.addJavascriptInterface(new CaulyJsInterface(testWebview),CaulyJsInterface.CAULY_JS_INTERFACE_NAME);

}

```

##### Get Platform String
Native SDK의 platform (Android / iOS) 값을 얻습니다. 리턴값은 ‘Android’ 또는 ‘iOS’ 입니다.
###### sample
```javascript
if(window.caulyJSInterface.platform() == 'Android'){
...
}else if(window.caulyJSInterface.platform() == 'iOS'){
...
}
```
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>This is Cauly Web</title>

<script type="text/javascript">
	window.onload = function() {
		if (window.caulyJSInterface) {
			var platform = window.caulyJSInterface.platform();
			document.getElementById('platform').innerText = platform;
		}
	}
</script>
</head>
<body>
	<div id="platform"></div>
	<a href="javascript:location.reload();">Reload</a>
</body>
</html>
```


##### Get Google Advertising ID
Android Play service에서 제공하는 Google Advertising ID를 얻습니다.

###### sample
테스트 웹페이지에 SDK를 통해 얻은 gaid를 출력합니다.
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>This is Cauly Web</title>

<script type="text/javascript">
	window.onload = function() {
		if (window.caulyJSInterface) {
			if(window.caulyJSInterface.platform() == "Android"){
				var gaid = window.caulyJSInterface.getAdId();
				document.getElementById('adid').innerText = gaid;
			}
		}
	}
</script>
</head>
<body>
	<div id="adid"></div>
	<a href="javascript:location.reload();">Reload</a>
</body>
</html>
```






