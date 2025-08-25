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



<br><br><br>

### ✅ 참고
<a href="https://developer.android.com/training/app-links?hl=ko">Android Developer, 다양한 유형의 링크 이해하기</a><br>
<a href="https://developer.android.com/guide/components/intents-filters?hl=ko#java">Android Developer, 인텐트 및 인텐트 필터</a>

#### [Applink]<br> 
<a href="https://developers.google.com/digital-asset-links/v1/getting-started?hl=ko">Google Digital Asset Links 동작 원리</a><br>
<a href="https://developer.android.com/training/app-links/verify-android-applinks?hl=ko&_gl=1*1hzn0af*_up*MQ..*_ga*MTg5MzQ2NDY3OS4xNzU0OTExNjU3*_ga_6HH9YJMN9M*czE3NTQ5MTE2NTckbzEkZzAkdDE3NTQ5MTE2NTckajYwJGwwJGg2NTk5MTgxMTI.#web-assoc">Android 앱 링크 인증하기</a>


#### [Deeplink] <br>
<a href="https://developer.android.com/training/app-links/deep-linking?hl=ko">Android Developer, Deep link 구현하기</a>