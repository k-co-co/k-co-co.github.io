---
layout: post
title: Deeplink
subtitle: 딥링크의 유형 그리고 취약점
categories: Mobile
tags: [Deep Link, App Link, Universal Link]
---

## 💥 딥링크(Deeplink)

<div style="display: flex; justify-content: center;">
  <table>
    <thead>
      <tr>
        <th style="text-align: center;">구분</th>
        <th style="text-align: center;">URI 스킴</th>
        <th style="text-align: center;">App Link</th>
        <th style="text-align: center;">Universal Link</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td style="text-align: center; white-space: nowrap;">설명</td>
        <td>커스텀 URI 스킴을 사용하여<br>앱 내 특정 화면으로 이동</td>
        <td>HTTP/HTTPS 기반 URL을 이용하며<br>Android에서 도메인 소유 인증을 거친 딥링크</td>
        <td>HTTP/HTTPS 기반 URL을 이용하며<br>iOS에서 도메인 소유 인증을 거친 딥링크</td>
      </tr>
      <tr>
        <td style="text-align: center; white-space: nowrap;">예시</td>
        <td>myapp://profile/123</td>
        <td>https://example.com/profile/123</td>
        <td>https://example.com/profile/123</td>
      </tr>
      <tr>
        <td style="text-align: center; white-space: nowrap;">플랫폼</td>
        <td>Android, iOS 모두</td>
        <td>Android 전용</td>
        <td>iOS 전용</td>
      </tr>
      <tr>
        <td style="text-align: center; white-space: nowrap;">보안성</td>
        <td>낮음 (스킴 하이재킹 가능)</td>
        <td>높음 (도메인 검증)</td>
        <td>높음 (도메인 검증)</td>
      </tr>
    </tbody>
  </table>
</div>

<br><br>
