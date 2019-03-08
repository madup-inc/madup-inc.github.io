---
layout: post
title: 토이 프로젝트로 슬랙 봇 만들기
author: Cindy(firepunch)
date: 2019-03-06
---

ㅋㅎㅋㅎ

---

**슬랙과 상호작용**
![흐름도](/images/2019/03/06/flow.png)
Permission Scopes
incoming-webhook

Incoming Webhooks
event cron 생성
lambda로 슬랙에 메세지 전송하기

Interactive Components
Request URL: 버튼과 상호작용하기

API Gateway

**슬랙으로 로그인**
OAuth
슬랙 로그인 Redirect URLs, 로그인 후 토큰 뭐시기

Permission Scopes
identity.basic

**구글 달력 연동**
server-to-server를 위해 JWT 사용

service account 생성
json으로 저장

https://developers.google.com/admin-sdk/directory/v1/guides/delegation
어드민 콘솔에서 클라이언트 아이디와 API 범위 지정


캘린더 공유
클라이언트 아이디를 공유함

테스트
https://developers.google.com/calendar/v3/reference/events/list

googleapis
https://github.com/googleapis/google-api-nodejs-client/blob/master/samples/jwt.js
auth, calendar.events.list 조회
