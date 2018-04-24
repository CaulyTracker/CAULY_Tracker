Cauly 리타겟팅 연동 가이드
=========================
Android Native APP
-------------------------- 
### 개요
네이티브 앱은 Java 로 작성된 일반적인 Android 앱을 지칭합니다. 본 문서는 광고주의 앱이 네이티브 앱일 경우 리타겟팅 연동을 하는 방법에 대해서 설명하는 문서입니다. 컨텐츠 제공에 Webview 를 주로 사용한다면 [Android Hybrid APP](https://github.com/CaulyTracker/Android-Tracking-SDK-Hybrid) 문서를 참고해주세요. 

### 문서 버전 
| 문서 버전 | 작성 날짜 | 작성자 및 내용 | 
| ---------- | ----------- | ---------------- |
| 1.0.0 | 2015.10.09 | 권대화(neilkwon@fsn.co.kr) - 초안작성 |
| 1.0.1 | 2016.04.06 | 권대화(neilkwon@fsn.co.kr) - 업데이트 내역 |
| 1.0.2 | 2016.04.28 | 권대화(neilkwon@fsn.co.kr) - 업데이트 내역 |
| 1.0.3 | 2016.05.16 | 권대화(neilkwon@fsn.co.kr) - 업데이트 내역(Purchase / ContentView(Product) Event 추가) |
| 1.0.4 | 2016.06.27 | 권대화(neilkwon@fsn.co.kr) - Android version 2.3 대응 |
| 1.0.5 | 2016.06.29 | 권대화(neilkwon@fsn.co.kr) - ContentView Bug Fix / Install referrer 로직보완 |
| 1.0.5 | 2016.07.04 | 권대화(neilkwon@fsn.co.kr) - 정밀성을 위한 추가 정보 전송 |
| 1.0.6 | 2016.09.22 | 권순국(nezy@fsn.co.kr) - 문서 레이아웃 변경 |

### 목차
- [연동 절차](#연동-절차)
- [연동 상세](#연동-상세)
  - [Native APP SDK 연동](#native-app-sdk-연동)
  - [Deep Link 처리 (해당시에만)](#deeplink-처리-해당시에만)
  - [Event 처리](#event-처리)
  - [Reference](#reference)

--------------------------

### 연동 절차

1. Cauly 담당자 혹은 fsn_rt@fsn.co.kr로 연락하여 Cauly 리타겟팅 연동에 대해서 협의합니다.
1. 협의 완료 후, Cauly 담당자를 통해 track_code를 발급받습니다.
1. track_code 를 포함한 tracker 코드를 아래 ‘연동상세’을 참고하여 해당하는 부분에 해당하는 곳에 삽입합니다.
1. 코드 삽입 이후 연동이 되었는지 Cauly에게 APK 전달하여 테스트를 요청합니다.
1. Cauly에서 테스트를 완료하면 APP을 마켓에 업데이트하고 업데이트 내용을 Cauly와 공유합니다.
1. 마켓 업데이트 완료 후 7일 뒤 (주말 및 공휴일 포함) 광고 라이브 가능합니다. (단, 모수가 너무 적을 경우 모수 수집을 위해 추가 시간이 소요될 수 있습니다.)

### 연동 상세
------------
#### Native APP SDK 연동
대상 OS 버전: Android 2.3 이상

| 항목 | 세부항목 | 목적 | 연동 가이드 |
| ---------- | -------------- | ----------- | --------------- |
| Setting | proguard | proguard 세팅 | [Proguard 부분 참고](#proguard) |
| | AndroidManifest.xml – track_code 추가 | track_code 추가 | [Initialize 부분 참고](#initialize) |
| Setting & Code | AndroidManifest.xml – Install referrer 추가 (option) | install 수 측정 | [Install referrer 부분 참고](#install-referrer-check) |
| 초기화 Code | Session | 앱 실행 측정 | [Session 부분 참고](#session) |

##### Proguard
Proguard 적용시에는 SDK에 적용되지 않도록 아래 설정을 추가
```
-keep class com.fsn.cauly.tracker.** { *; }
```

##### Initialize
AndroidManifest.xml 내에 meta data 태그로 발급받은 track code를 입력합니다. 
예시의 '[CAULY_TRACK_CODE]'부분을 변경합니다. ( [] 기호는 불필요 )
```xml
<application
...
 <meta-data
	android:name="cauly_track_code"
	android:value="[CAULY_TRACK_CODE]" />

...
</application>
```

##### Session
Tracking을 시작하는 시점과 종료하는 시점에 호출. CaulyTrackerBuilder를 통해 tracker instance를 초기화 하는 단계에서 ‘start’를 호출하는 것이 권장됩니다.

###### sample
startSession
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	setContentView(R.layout.activity_main);

	CaulyTrackerBuilder caulyTrackerBuilder = new CaulyTrackerBuilder(getApplicationContext());

	CaulyTracker caulyTracker = caulyTrackerBuilder.setUserId("customer_id_0922451")
			.setAge("25")
			.setGender(TrackerConst.FEMALE)
			.setLogLevel(LogLevel.Debug)
			.build();
	caulyTracker.startSession();
}
```
또는  Android Activity LifeCycle 중 Activity가 Active 상태가 될때 Callback을 받는 overrided method중에 한 곳에 작성합니다.

```java
@Override
protected void onResume() {
	super.onResume();
	try {
		CaulyTrackerBuilder.getTrackerInstance().startSession();
	} catch (CaulyException e) {
		e.printStackTrace();
	}
}
```

close
```java
@Override
protected void onDestroy() {
	super.onDestroy();
	try {
		CaulyTrackerBuilder.getTrackerInstance().closeSession();
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```

##### Install Referrer Check
앱 인스톨 시점에 설치 정보를 전송합니다.
Install Referer를 사용하는 방법은 아래의 2가지 형태가 있습니다. ( 하나의 방식만 구현)
 
- XML - CaulyTracker를 단독 Install referrer receiver 사용할때
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.neilkwon.fsntrackertester"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk
        android:minSdkVersion="8"
        android:targetSdkVersion="21" />
    <uses-permission android:name="android.permission.INTERNET"/>

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <receiver android:name="com.fsn.cauly.tracker.InstallReceiver">
            <intent-filter>
                <action android:name="com.android.vending.INSTALL_REFERRER"/>
            </intent-filter>
        </receiver>

        <meta-data
            android:name="cauly_track_code"
            android:value="NEIL_TEST_CODE" />

    </application>

</manifest>
```

Source - 다수의 Install referrer receiver를 사용할 때
- INSTALL_REFERRER broadcast receiver로 등록된 Class 에서 아래와 같이 추가.

```java
import com.fsn.cauly.tracker.InstallReceiver;

public class OtherInstallReceiver extends BroadcastReceiver {

	public OtherInstallReceiver() {

	}

	@Override
	public void onReceive(Context context, Intent intent) {
		if (intent.getAction().equals("com.android.vending.INSTALL_REFERRER")) {
			String referrerStr = intent.getStringExtra("referrer");
			if (referrerStr != null) {
				try {

					//Cauly Install check
					// 간단한 설치 정보만 전송
					InstallReceiver installReceiver = new InstallReceiver();
					installReceiver.onReceive(context, intent);

					// Other tracker 1 - 타사SDK
					InstallReceiverEtc1  installReceiverEtc1 = new InstallReceiverEtc1();
					installReceiverEtc1.onReceive(context, intent);

					// Other tracker 2 - 타사SDK
					InstallReceiverEtc2 installReceiverEtc2 = new InstallReceiverEtc2();
					installReceiverEtc2.onReceive(context, intent);

				} catch (Exception e) {

				}
			}
		}
	}

}

```


#### DeepLink 처리 (해당시에만)
광고주의 APP이 Deep Link를 지원하여 유저가 광고를 클릭 했을 때 랜딩하는 위치가 APP의 메인 페이지가 아닌 다른 특정 페이지 (또는 상품상세페이지)인 경우에만 해당되는 사항입니다. 해당사항이 없을 경우 Event 처리 단계로 넘어갑니다.
```java
// 아래 String 처럼 DeekLink 가 들어왔다고 가정하면
// String deepLinkStr = "someapp://app/?cauly_rt_code=1234&cauly_egmt_sec=8600"; 
CaulyTracker caulyTracker = CaulyTrackerBuilder.getTrackerInstance();
caulyTracker.traceDeepLink(deepLinkStr);
```

#### Event 처리
아래 2가지 캠페인 중 집행 예정인 캠페인에 맞게 code 를 삽입합니다.
- [A. Feed 캠페인](#a-feed-캠페인)
- [B. Static 캠페인](#b-static 켐페인)

#### A.	Feed 캠페인
| 이벤트명 | 목적 | 연동 가이드 |
| -------- | ----- | --------- |
| OPEN | - 리타겟팅 광고 노출 대상자 선정 | [OPEN 이벤트](#open-이벤트) |
| PRODUCT | - 광고노출 대상자 별 추천 상품목록 생성 | [상품 VIEW 이벤트](#상품-view-이벤트) |
| CART | - 광고노출 대상자 별 추천 상품목록 생성(option)  | [장바구니 이벤트](#장바구니-이벤트) |
| SEARCH | - 사용자의 검색어 (option) | [검색 이벤트](#검색-이벤트) |
| CONTENT | - 상품이미지 및 상품 상세정보를 광고 소재로 활용 | [ContentView 이벤트](#contentview-이벤트) |
| PURCHASE | - 추천상품에서 구매상품 제외 처리 <br>- ROAS 측정 | [PURCHASE 이벤트](#purchase-이벤트) |
| RE-PURCHASE | - 재구매율 측정 (option) | [RE-PURCHASE 이벤트](#re-purchase-이벤트) |

#### B. Static 캠페인
| 이벤트명 | 목적 | 연동 가이드 |
| -------- | ----- | --------- |
| OPEN | - 리타겟팅 광고 노출 대상자 선정 | [OPEN 이벤트](#open-이벤트) |
| CA_CONVERSION | - 전환 건수 측정 <br>- 예) 상담신청완료 등 | [CONVERSION 이벤트](#conversion-이벤트) |

##### OPEN 이벤트
```java
CaulyTracker caulyTracker = CaulyTrackerBuilder.getTrackerInstance();
caulyTracker.trackEvent("OPEN");
```

##### 상품 view 이벤트
```java
String productId = "987654321"; // 광고주의 product id 를 987654321 라 가정하면
CaulyTracker caulyTracker = CaulyTrackerBuilder.getTrackerInstance();
caulyTracker.trackEvent("PRODUCT", productId);
```

##### 검색 이벤트
```java
String searchWord = "SEARCH_WORD"; // 사용자의 검색어
CaulyTracker caulyTracker = CaulyTrackerBuilder.getTrackerInstance();
caulyTracker.trackEvent("SEARCH", searchWord);
```

##### 장바구니 이벤트
```java
String productId = "987654321"; // 광고주의 product id 를 987654321 라 가정하면
CaulyTracker caulyTracker = CaulyTrackerBuilder.getTrackerInstance();
caulyTracker.trackEvent("CART", productId);
```

##### ContentView 이벤트
```java
final String itemId = "p20160510_test_1";
final String itemName = "[오늘의 특가] 카울리 반창고!";
final String itemImage = "https://www.cauly.net/images/logo_cauly_main.png";
final String itemUrl = "caulytrackertest://caulytracker.com/product?item_id=p20160510_test_1";
final String originalPrice = "24000";
final String salePrice = "18000";
final String category1 = "생활물품";
final String category2 = "구급";

final String category3 = "";
final String category4 = "";
final String category5 = "";
final String regDate = "";
final String updateDate = "";
final String expireDate = "";
final String stock = "10";
final String state = "available";
final String description = " 한번 사용하면 멈출 수 없는 쫄깃함 !";
final String extraImage = "";
final String locale = "KRW";


CaulyTrackerContentViewEvent caulyTrackerContentViewEvent = new CaulyTrackerContentViewEvent(
								itemId);

caulyTrackerContentViewEvent.setItemImage(itemImage);
caulyTrackerContentViewEvent.setItemName(itemName);
caulyTrackerContentViewEvent.setItemUrl(itemUrl);
caulyTrackerContentViewEvent.setOriginalPrice(originalPrice);
caulyTrackerContentViewEvent.setSalePrice(salePrice);
caulyTrackerContentViewEvent.setCategory1(category1);
caulyTrackerContentViewEvent.setCategory2(category2);

// Optional info.
CaulyTrackerContentViewEvent.ContentDetail detail = new CaulyTrackerContentViewEvent.ContentDetail();

detail.setCategory3(category3);
detail.setCategory4(category4);
detail.setCategory5(category5);
detail.setRegDate(regDate);
detail.setUpdateDate(updateDate);
detail.setExpireDate(expireDate);
detail.setStock(stock);
detail.setState(state);
detail.setDescription(description);
detail.setExtraImage(extraImage);
detail.setLocale(locale);

caulyTrackerContentViewEvent.setDetail(detail);

try {
	CaulyTrackerBuilder.getTrackerInstance().trackEvent(caulyTrackerContentViewEvent);
} catch (CaulyException e) {
	e.printStackTrace();
}

```

##### PURCHASE 이벤트
```java
// 유저가 구매한 20000원짜리 (product id "987654321") 3개와 10000원짜리 (product id "887654321") 1개를 샀고,
// 그래서 총 구매액은 70000원이고,
// 광고주가 발급한 구매 id(order id) 가 "order_20160430" 라고 가정하면,
CaulyTrackerPurchaseEvent purchaseEvent = new CaulyTrackerPurchaseEvent();

String productId = "987654321";
String productPrice = "20000";
String productQuantity = "3";
Product product = new Product(productId, productPrice, productQuantity);

String productId2 = "887654321";
String productPrice2 = "10000";
String productQuantity2 = "1";
Product product2 = new Product(productId2, productPrice2, productQuantity2);

purchaseEvent.setOrderId("order_20160430");
purchaseEvent.setOrderPrice("70000");
purchaseEvent.addProuduct(product);
purchaseEvent.addProuduct(product2);
purchaseEvent.setCurrencyCode(TrackerConst.CURRENCY_KRW);

CaulyTrackerBuilder.getTrackerInstance().trackEvent(purchaseEvent);
```

##### Re-Purchase 이벤트
재구매(첫 구매가 아닌) 유저를 골라서 분류해보고 싶으면 아래처럼 한 줄 추가된 코드를 사용합니다.
```java
CaulyTrackerPurchaseEvent purchaseEvent = new CaulyTrackerPurchaseEvent();

String productId = "p_0344411";
String productPrice = "20000";
String productQuantity = "3";
Product product = new Product(productId, productPrice, productQuantity);

String productId2 = "p_0344412";
String productPrice2 = "10000";
String productQuantity2 = "1";
Product product = new Product(productId2, productPrice2, productQuantity2);

// 아래 한 줄을 추가합니다
purchaseEvent.setPurchaseType("RE-PURCHASE");
// 한 줄 추가 끝
purchaseEvent.setOrderId("order_20160430");
purchaseEvent.setOrderPrice("70000");
purchaseEvent.addProuduct(product);
purchaseEvent.addProuduct(product2);
purchaseEvent.setCurrencyCode(TrackerConst.CURRENCY_KRW);

CaulyTrackerBuilder.getTrackerInstance().trackEvent(purchaseEvent);
```

##### Conversion 이벤트
```java
CaulyTracker caulyTracker = CaulyTrackerBuilder.getTrackerInstance();
caulyTracker.trackEvent("CA_CONVERSION");
```

---------------------

#### Reference

##### SDK 구조
###### CaulyTrackerBuilder
기본적인 설정 정보와 함께 CaulyTracker instance를 초기화합니다.
<br>
![CaulyTracker class](misc/CaulyTrackerBuilder.png)


###### CaulyTracker
Event를 Tracking 할 수 있는 method들을 제공합니다.
<br>
![CaulyTracker class](misc/CaulyTracker.png)


##### CaulyTrackerBuilder를 통한 초기화
| Method | mandatory | Description |
| --------- | ------------- | ------------- |
| setUserId | optional | 각 서비스를 사용하는 사용자의 고유 ID |
| setAge | optional | 사용자의 연령<br>연령 정보를 추가하면 더욱 세밀한 분석이 가능합니다.|
| setGender | optional | 사용자의 성별<br>성별 정보를 추가하면 더욱 세밀한 분석이 가능합니다. |
| setLogLevel | optional | 로그 출력 level<br>default : Info
| build | mandatory | tracker instance를 생성 |

###### Sample
```java
CaulyTrackerBuilder caulyTrackerBuilder = new CaulyTrackerBuilder(getApplicationContext());

caulyTracker = caulyTrackerBuilder.setUserId("customer_id_0922451")
				.setAge("25")
				.setGender(TrackerConst.FEMALE)
				.setLogLevel(LogLevel.Debug)
				.build();
```

userId, Age, Gender등의 정보는 Builder 로 초기화한 이후 CaulyTracker instance를 통해서도 변경가능합니다.

```java
try {
	CaulyTrackerBuilder.getTrackerInstance().setUserId("user_001");
	...
	
	CaulyTrackerBuilder.getTrackerInstance().setAge("20");
	...
	CaulyTrackerBuilder.getTrackerInstance().setGender(TrackerConst.FEMALE);
	...
} catch (CaulyException e) {
	// CaulyTracker Instance not initialized.
}
```

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

##### Event

###### Custom Event
| Parameter | Required | Description |
|-----------|----------|-------------|
| event_name | mandatory | 트래킹할 이벤트명 |
| event_param | optional | 세부 정보 등 이벤트에 추가적으로 기입할 값 |


####### name only sample
```java
caulyTracker.trackEvent("event1_nameonly");
```
####### name / single param sample
```java
caulyTracker.trackEvent("event2_string", "test");
```

###### Defined Event
자주 사용되거나 또는 중요하다 판단되는 Event에 대한 선정의된 Event입니다.

####### Purchase
구매 또는 지불이 발생하였을때 호출

| Parameter | Type |Required | Default | Description |
| --------- | ---- | ------- | ------- | ----------- |
| order_id | String | mandatory | - | Order ID |
| order_price | String | mandatory | - | 발생한 전체 금액 |
| purchase_type | String | optional | - | 구매의 성격<br>eg)재구매 : RE-PURCHASE |
| product_infos | List<Product> | mandatory | - | 구매된 상품의 상세 정보 목록<br>최소 1개 이상 상품이 등록되어야 합니다. |
| currency_code | String | optional | KRW | 통화 코드 |

```java
CaulyTrackerPurchaseEvent purchaseEvent = new CaulyTrackerPurchaseEvent();

String productId = "p_0344411";
String productPrice = "20000";
String productQuantity = "3";
Product product = new Product(productId, productPrice, productQuantity);

String productId2 = "p_0344412";
String productPrice2 = "10000";
String productQuantity2 = "1";
Product product = new Product(productId2, productPrice2, productQuantity2);

purchaseEvent.setOrderId("order_20160430");
purchaseEvent.setOrderPrice("70000");
purchaseEvent.addProuduct(product);
purchaseEvent.addProuduct(product2);
purchaseEvent.setCurrencyCode(TrackerConst.CURRENCY_KRW);

CaulyTrackerBuilder.getTrackerInstance().trackEvent(purchaseEvent);
```

####### ContentView(Product)
Content(Product)에 대한 트래킹
상품의 상세한 정보가 포함된 이벤트입니다.

```java
final String itemId = "p20160510_test_1";
final String itemName = "[오늘의 특가] 카울리 반창고!";
final String itemImage = "https://www.cauly.net/images/logo_cauly_main.png";
final String itemUrl = "caulytrackertest://caulytracker.com/product?item_id=p20160510_test_1";
final String originalPrice = "24000";
final String salePrice = "18000";
final String category1 = "생활물품";
final String category2 = "구급";

final String category3 = "";
final String category4 = "";
final String category5 = "";
final String regDate = "";
final String updateDate = "";
final String expireDate = "";
final String stock = "10";
final String state = "available";
final String description = " 한번 사용하면 멈출 수 없는 쫄깃함 !";
final String extraImage = "";
final String locale = "KRW";


CaulyTrackerContentViewEvent caulyTrackerContentViewEvent = new CaulyTrackerContentViewEvent(
								itemId);

caulyTrackerContentViewEvent.setItemImage(itemImage);
caulyTrackerContentViewEvent.setItemName(itemName);
caulyTrackerContentViewEvent.setItemUrl(itemUrl);
caulyTrackerContentViewEvent.setOriginalPrice(originalPrice);
caulyTrackerContentViewEvent.setSalePrice(salePrice);
caulyTrackerContentViewEvent.setCategory1(category1);
caulyTrackerContentViewEvent.setCategory2(category2);

// Optional info.
CaulyTrackerContentViewEvent.ContentDetail detail = new CaulyTrackerContentViewEvent.ContentDetail();

detail.setCategory3(category3);
detail.setCategory4(category4);
detail.setCategory5(category5);
detail.setRegDate(regDate);
detail.setUpdateDate(updateDate);
detail.setExpireDate(expireDate);
detail.setStock(stock);
detail.setState(state);
detail.setDescription(description);
detail.setExtraImage(extraImage);
detail.setLocale(locale);

caulyTrackerContentViewEvent.setDetail(detail);

try {
	CaulyTrackerBuilder.getTrackerInstance().trackEvent(caulyTrackerContentViewEvent);
} catch (CaulyException e) {
	e.printStackTrace();
}

```





