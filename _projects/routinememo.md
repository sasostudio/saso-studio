---
layout: project
title: 루틴메모
headline: 내 불편에서 바로 만들어 직접 쓰는 운동 기록 앱
status: LIVE
dim: false
order: 2
blurb: 메모장 루틴과 운동 앱을 오가던 제작자 본인의 불편에서 출발한 Android 운동 기록 앱. 직접 만들어 실사용 중, Google Play 정식 출시.
meta: [Android, Kotlin, Jetpack Compose, Firebase, 2026]
steps:
  - k: "문제 / Problem"
    h: 메모장 루틴과 운동 앱 사이의 이중 기록
    p: 운동 루틴은 메모장에 정리, 기록은 운동 앱에 따로 입력하던 방식. 같은 내용을 두 번 옮겨 적는 번거로움에, 기존 앱으로는 메모장식 표기(순서·대안 종목·메모)를 그대로 담기 어려운 문제까지 존재.
  - k: "해결 / What"
    h: 적어 둔 루틴이 그대로 오늘의 체크리스트
    p: 메모장 표기 그대로 루틴을 붙여넣으면 체크리스트로 변환, 운동 중에는 세트마다 무게·횟수·버티기 시간 기록. 화면을 꺼도 동작하는 휴식 타이머와 그날 운동의 텍스트 공유까지 포함. 백그라운드 타이머 정확도 때문에 웹뷰 대신 Kotlin·Jetpack Compose 네이티브 채택.
  - k: "현황 / Status"
    h: 제작자 본인의 실사용 중, Google Play 정식 출시
    p: 필요하다고 느낀 시점에 바로 제작해 실제 운동 기록 도구로 계속 사용 중. 쓰면서 걸리는 점은 바로 수정해 배포하는 방식으로 개선(예 — 분·초 입력 칸 분리, 붙여넣은 종목 이름의 괄호 유지). 개인용이지만 스토어 정책 요건(계정 삭제·개인정보처리방침·연령 확인)까지 갖춰 2026년 10월 27일 Google Play 정식 출시(대한민국). 무료, 광고 미포함.
play:
  - { label: "Google Play", url: "https://play.google.com/store/apps/details?id=com.saso.routinelog" }
links:
  - { label: "개인정보처리방침", url: "/routinememo/privacy/" }
---
