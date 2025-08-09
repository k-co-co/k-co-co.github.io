---
layout: post
title: Applink vs Deeplink
subtitle: 앱링크와 딥링크의 차이, 그리고 취약점
categories: Mobile
tags: [앱링크, 딥링크]
---

## 💥 앱링크(Applink) vs 딥링크(Deeplink)

<div style="display: flex; justify-content: center;">
  <table>
    <thead>
      <tr>
        <th style="text-align: center;">구분</th>
        <th style="text-align: center;">Deep Link</th>
        <th style="text-align: center;">App Link</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td style="text-align: center; white-space: nowrap;">설명</td>
        <td>커스텀 URI 또는 일반 딥링크 URL로 앱 내 특정 화면으로 이동</td>
        <td>HTTP/HTTPS 기반 URL을 이용하며 도메인 소유 인증을 거친 딥링크 형태</td>
      </tr>
      <tr>
        <td style="text-align: center; white-space: nowrap;">스킴</td>
        <td>커스텀 스킴(ex. myapp://)</td>
        <td>HTTP 또는 HTTPS URL</td>
      </tr>
      <tr>
        <td style="text-align: center; white-space: nowrap;">보안</td>
        <td>낮음(별도의 인증 없음)</td>
        <td>높음(도메인 소유권 확인)</td>
      </tr>
      <tr>
        <td style="text-align: center; white-space: nowrap;">비고</td>
        <td>-</td>
        <td>Android: App Link / IOS: Universal Link</td>
      </tr>
    </tbody>
  </table>
</div>

<br><br><br>

### App Link
App Link부터 살펴보자.


### Deep Link