---
layout: post
title: Deeplink
subtitle: 딥링크의 유형 그리고 취약점
categories: Mobile
tags: [Deep Link, App Link, Universal Link]
---

### ✅ Deeplink의 유형

<div style="display: flex; justify-content: center;">
  <table style="table-layout: fixed; width: 100%; max-width: 800px;">
    <thead>
      <tr>
        <th style="text-align: center; width: auto;">구분</th>
        <th style="text-align: center; width: 33.33%;">URI 스킴</th>
        <th style="text-align: center; width: 33.33%;">App Link</th>
        <th style="text-align: center; width: 33.33%;">Universal Link</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td style="text-align: center; white-space: nowrap;">설명</td>
        <td>커스텀 URI 스킴을 사용하여 앱 내 특정 화면으로 이동</td>
        <td>HTTP/HTTPS 기반 URL을 이용하며 Android에서 도메인 소유 인증을 거친 딥링크</td>
        <td>HTTP/HTTPS 기반 URL을 이용하며 IOS에서 도메인 소유 인증을 거친 딥링크</td>
      </tr>
      <tr>
        <td style="text-align: center; white-space: nowrap;">예시</td>
        <td>coco://k-co-co/</td>
        <td>https://android.com/k-co-co/</td>
        <td>https://ios.com/k-co-co/</td>
      </tr>
      <tr>
        <td style="text-align: center; white-space: nowrap;">플랫폼</td>
        <td>Android, iOS</td>
        <td>Android</td>
        <td>iOS</td>
      </tr>
      <tr>
        <td style="text-align: center; white-space: nowrap;">보안성</td>
        <td>낮음(별도의 검증 없음)</td>
        <td>높음(도메인 검증)</td>
        <td>높음(도메인 검증)</td>
      </tr>
    </tbody>
  </table>
</div>

<br>

💡 App Link와 URI 스킴을 살펴보려고 한다.

#### Applink
> 앱 내의 특정 페이지로 바로 연결시켜주는 딥링크로, HTTP/HTTPS URL을 사용한다.

사용자의 스마트폰이 해당 앱을 가지고 있다면 앱의 특정 콘텐츠를 바로 띄워주며, 만약 설치되어 있지 않다면 웹 페이지로 콘텐츠를 띄어주게 된다.
(* Android 6.0, Api 23 이상)

<br>

우선 앱링크를 구현하려면 AndroidManifest.xml 파일 내 아래와 같이 선언되어야 한다. <br>

<summary>Android Developer 발췌</summary>

```xml
  <intent-filter android:autoVerify="true">
      <action android:name="android.intent.action.VIEW" />
      <category android:name="android.intent.category.DEFAULT" />
      <category android:name="android.intent.category.BROWSABLE" />

      <!-- Do not include other schemes. -->
      <data android:scheme="http" />
      <data android:scheme="https" />
      <data android:host="k-coco.github.com"/>
  </intent-filter>
```

<br>
intent-filter 내에 다음과 같이 AutoVerify 속성을 true로 설정해줘야한다.
AutoVerify 속성을 통해 해당 앱이 실제로 정의된 도메인과 연결된 것인지 시스템으로부터 검증하도록 요청한다.


<br>
다음으로는 운영중인 웹 서버에 assetlinks.json을 게시한다. assetslink 파일 내용은 아래와 같다.
해당 파일에서 중요하게 볼 부분은 두 가지이다. 바로 **package_name**과 **sha256_cert_fingerprints**이다.



<summary>Google 발췌</summary>

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "coco_app",
    "package_name": "k-coco.github",
    "sha256_cert_fingerprints":
    ["14:6D:E9:83:C5:73:06:50:D8:EE:B9:95:2F:34:FC:64:16:A0:83:42:E6:1D:BE:A8:8A:04:96:B2:3F:CF:44:E5"]
  }
}]
```


위와 같은 설정이 적용된 상태에서 어떻게 동작하는지 살펴보면 아래와 같다.

1. 사용자가 앱을 설치한다. <br>
1-1. 앱 설치 시 Intent-filter 내에 선언된 autoVerify 속성을 확인하고, 해당 도메인에 있는 assetslinks에 접근한다. <br>
1-2. assetslinks 내에 있는 SHA256 인증서 해시값을 통해 도메인과 앱이 매핑되는지 검증하고, 검증에 성공하면 시스템(Package Manager) 내에 매핑정보를 저장한다. <br>

2. 사용자가 해당 도메인을 클릭할 때 마다 매핑된 앱을 실행시켜 보여준다.

<br>
ps. 검증 여부는 /data/system/packages.xml 내에 가지고 있는 것 같다.(Android 9.0 기준)
<br><br><br>

___

#### URI Scheme
> 앱 내의 특정 페이지로 바로 연결시켜주는 딥링크로, 커스텀 URI을 사용한다.

URI Scheme 방식은 앱링크에 비해 간단하다. AndroidManifest.xml 내에 아래와 같이 선언만 해주면 된다.

<summary>Google 발췌</summary>
```xml
<activity
    android:name="com.coco.github.MainActivity"
    android:label="@string/title_main" >
    <intent-filter android:label="@string/filter_view_http_coco_github">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- Accepts URIs that begin with "http://github.io/first" -->
        <data android:scheme="http"
              android:host="github.io"
              android:pathPrefix="/first" />
    </intent-filter>
    <intent-filter android:label="@string/filter_view_coco_github">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- Accepts URIs that begin with "coco://github.io" -->
        <data android:scheme="coco"
              android:host="github.io" />
    </intent-filter>
</activity>
```

두 번째 Intent-filter를 보면 된다. <br>
scheme은 coco, host는 github.io이므로 **_coco://github.io_** 형태로 호출하게 된다.
해당 예시는 scheme과 host만 구현되어있지만 이 외에도 path와 parameter를 명시하여 받을 수 있다.


<br>


### ✅ Deeplink 취약점

> 23년도 금취분평 체크리스트에 추가된 취약점으로 딥링크를 조작하여 앱의 의도하지 않은 기능이 실행되는 취약점

개발자가 딥링크를 받아 어떻게 처리하는지에 따라 다르지만 해당 취약점이 존재할 경우 <span style="color: red">인증 우회, 임의 페이지 리다이렉션, JavaScript 코드 실행과 같은 공격</span>이 가능하다.


### ✅ Deeplink 취약점 시연
<a href="https://github.com/hax0rgb/InsecureShop">Insecureshop</a>이라는 오픈소스 APK가 존재한다.<br>
해당 앱은 Android 앱에서 발생할 수 있는 취약점들을 학습하기 위해 만들어졌다.

Jadx로 해당 앱을 디컴파일하여 AndroidManifest.xml부터 살펴본다.
![Android Manifest 분석]({{ '/assets/images/source/250809_img/manifest.png' | relative_url }})
WebViewActivity를 <span style="color: red">insecureshop://com.insecureshop</span> 형태로 호출이 가능하다.

<br>


이제 WebViewActivity로 이동해서 OnCreate부터 살펴본다. Jadx에서 Ctrl + 우클릭을 통해 WebViewActivity로 바로 이동할 수 있다. ㄱㄱ <br>
WebviewActivity를 전체 해석할 필요는 없고, 액티비티 실행 시 호출되는 OnCreate 메서드만 살펴본다.

```Java
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
  
                     ... 중략 ...
        Intent intent = getIntent(); // WebViewActivity를 실행시킨 Intent 저장.
        Intrinsics.checkExpressionValueIsNotNull(intent, "intent"); // Null 체크
        Uri uri = intent.getData(); /* 
            intent.getData를 통해 딥링크에 전달된 Data를 uri에 저장. 
            ex. uri = insecureshop://com.insecureshop/web?url=https://k-co-co.github.io/
        */
        if (uri != null) { 
            String data = (String) null; // data 객체 생성
            if (!StringsKt.equals$default(uri.getPath(), "/web", false, 2, null)) {
                // uri.getPath를 통해 com.insecureshop 뒤의 /web를 첫 번째 인자
                // web과 동일하면 else문으로 이동
                // web이 아니면 !false -> true로 아래의 if문으로 이동
                if (StringsKt.equals$default(uri.getPath(), "/webview", false, 2, null)) {
                  // /webview라면 아래의 코드 실행
                    Intent intent2 = getIntent();
                    Intrinsics.checkExpressionValueIsNotNull(intent2, "intent");
                    Uri data2 = intent2.getData();
                    if (data2 == null) { // null일 경우
                        Intrinsics.throwNpe();
                    }
                    String queryParameter = data2.getQueryParameter("url"); // url파라미터값을 queryParameter에 저장
                    if (queryParameter == null) {
                        Intrinsics.throwNpe();
                    }
                    Intrinsics.checkExpressionValueIsNotNull(queryParameter, "intent.data!!.getQueryParameter(\"url\")!!");
                    if (StringsKt.endsWith$default(queryParameter, "insecureshopapp.com", false, 2, (Object) null)) {
                      // url의 쿼리파라미터값이 insecureshopapp.com으로 끝나야함.(endsWith Method)
                        Intent intent3 = getIntent();
                        Intrinsics.checkExpressionValueIsNotNull(intent3, "intent");
                        Uri data3 = intent3.getData();
                        data = data3 != null ? data3.getQueryParameter("url") : null; // data에 url 파라미터값 저장.
                    }
                }
            } else {
                Intent intent4 = getIntent();
                Intrinsics.checkExpressionValueIsNotNull(intent4, "intent");
                Uri data4 = intent4.getData();
                data = data4 != null ? data4.getQueryParameter("url") : null;
                // url 파라미터값을 data에 저장
            }
            if (data == null) { // data가 null이면 아무것도 하지않고 종료
                finish();
            }
            webview.loadUrl(data); // null이 아닌 경우 data를 url로 웹 뷰 실행
            Prefs.INSTANCE.getInstance(this).setData(data);
        }
    }


```

코드에 대한 해석은 주석을 확인하면 된다. <br>
결론적으로 insecureshop://com.insecureshop 딥링크를 실행시킬 때, uri Path를 /web or /webview로 주면 딥링크 취약점이 발생한다.

1) uri path가 /web인 경우, insecureshop://com.insecureshop/web?url=https://k-co-co.github.io/ 
```
adb shell am start -W -a android.intent.action.VIEW -d "insecureshop://com.insecureshop/web?url=https://k-co-co.github.io/"
```
<br>

2) uri path가 /webview인 경우, insecureshop://com.insecureshop/web?url=https://k-co-co.github.io/?data=insecureshopapp.com (endsWith Method)
```
adb shell am start -W -a android.intent.action.VIEW -d "insecureshop://com.insecureshop/web?url=https://k-co-co.github.io/?data=insecureshopapp.com"
```

<br><br>

위와 같이 호출할 경우, url 파라미터에 있는 URL을 WebView에서 실행하게 된다.
![Android Manifest 분석](/assets/images/source/250809_img/mobile.png)
블로그를 InsecureShop의 WebView로 띄운 모습이다.

<br>

나중에 더 추가해보도록 하자...

<br><br><br>

### ✅ 참고
<a href="https://developer.android.com/training/app-links?hl=ko">Android Developer, 다양한 유형의 링크 이해하기</a><br>
<a href="https://developer.android.com/guide/components/intents-filters?hl=ko#java">Android Developer, 인텐트 및 인텐트 필터</a>

#### [Applink]<br> 
<a href="https://developers.google.com/digital-asset-links/v1/getting-started?hl=ko">Google Digital Asset Links 동작 원리</a><br>
<a href="https://developer.android.com/training/app-links/verify-android-applinks?hl=ko&_gl=1*1hzn0af*_up*MQ..*_ga*MTg5MzQ2NDY3OS4xNzU0OTExNjU3*_ga_6HH9YJMN9M*czE3NTQ5MTE2NTckbzEkZzAkdDE3NTQ5MTE2NTckajYwJGwwJGg2NTk5MTgxMTI.#web-assoc">Android 앱 링크 인증하기</a>


#### [Deeplink] 
<a href="https://developer.android.com/training/app-links/deep-linking?hl=ko">Android Developer, Deep link 구현하기</a><br>
<a href="https://developer.android.com/privacy-and-security/risks/unsafe-use-of-deeplinks?hl=ko">Android Developer, 딥링크 취약점</a><br>
<a href="https://developer.android.com/reference/android/net/Uri#getPath()">Android Developer, Uri 객체</a><br>