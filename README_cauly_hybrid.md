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
	testWebview.getSettings().setDomStorageEnabled(true);
	testWebview.getSettings().setSupportMultipleWindows(true);
	
	testWebview.setWebChromeClient(new WebChromeClient() {
		@Override
		public boolean onJsAlert(final WebView view, final String url, final String message, JsResult result) {
			Log.d(TAG, "onJsAlert(!" + view + ", " + url + ", " + message + ", " + result + ")");
			Toast.makeText(getApplicationContext(), message, 3000).show();
			result.confirm();
			return true;
		}
		 // 새창열기시, 외부브라우져로 호출
		 @Override
		 public boolean onCreateWindow(WebView view, boolean isDialog,  boolean isUserGesture, Message resultMsg) {
		    	try{
				if(isUserGesture){
					 WebView newWebView = new WebView(view.getContext());
					    view.addView(newWebView);
					    WebView.WebViewTransport transport = (WebView.WebViewTransport) resultMsg.obj;
					    transport.setWebView(newWebView);
					    resultMsg.sendToTarget();
					    newWebView.setWebViewClient(new WebViewClient() {
						@Override
						public boolean shouldOverrideUrlLoading(WebView view, String url) {
							 Intent browserIntent = new Intent(Intent.ACTION_VIEW);
						      browserIntent.setData(Uri.parse(url));
						      startActivity(browserIntent);
							return true;
						}
					    });
				}
			} catch (Exception e) {
			}
		        return true;
		    }
	});
	testWebview.setWebViewClient(new WebViewClient(){
        	@Override
    		public boolean shouldOverrideUrlLoading(WebView view, String url) {
    			   if ( view == null || url == null) {
    	                // 처리하지 못함
    	                return false;
    	            }

    	            if ( url.contains("play.google.com") ) {
    	              // play.google.com 도메인이면서 App 링크인 경우에는 market:// 로 변경
    	              String[] params = url.split("details");
    	              if ( params.length > 1 ) {
    	                  url = "market://details" + params[1];
    	                  view.getContext().startActivity(new Intent(Intent.ACTION_VIEW,Uri.parse(url) ));
    	                  return true;
    	              }
    	            }

    	            if ( url.startsWith("http:") || url.startsWith("https:") ) {
    	                // HTTP/HTTPS 요청은 내부에서 처리한다.
    	                view.loadUrl(url);
    	            } else {
    	                Intent intent;

    	                try {
    	                    intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME);
    	                } catch (URISyntaxException e) {
    	                    // 처리하지 못함
    	                    return false;
    	                }

    	                try {
    	                    view.getContext().startActivity(intent);
    	                } catch (ActivityNotFoundException e) {
    	                    // Intent Scheme인 경우, 앱이 설치되어 있지 않으면 Market으로 연결
    	                    if ( url.startsWith("intent:") && intent.getPackage() != null) {
    	                        url = "market://details?id=" + intent.getPackage();
    	                        view.getContext().startActivity(new Intent(Intent.ACTION_VIEW,Uri.parse(url) ));
    	                        return true;
    	                    } else {
    	                        // 처리하지 못함
    	                        return false;
    	                    }
    	                }
    	            }
    	            return true;
    		}
        });
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






