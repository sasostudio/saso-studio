---
layout: project
title: 루틴메모
headline: 메모장처럼 적은 루틴을 그대로 수행하는 운동 기록 앱
status: LIVE
dim: false
order: 2
blurb: 메모장에 적던 운동 루틴을 붙여넣으면 체크리스트로 바뀌는 Android 운동 기록 앱. 세트·무게·시간 기록과 휴식 타이머 포함, Google Play 정식 출시.
meta: [Android, Kotlin, Jetpack Compose, Firebase, 2026]
steps:
  - k: "무엇 / What"
    h: 적어 둔 루틴이 곧 오늘의 체크리스트
    p: 메모장에 적던 운동 루틴을 텍스트 그대로 붙여넣으면 종목·세트·목표가 체크리스트로 변환. 운동 중에는 세트마다 무게·횟수·버티기 시간을 기록하고, 휴식 타이머는 화면을 꺼도 알림으로 동작. 그날 운동은 텍스트로 뽑아 SNS 공유 가능. 처음 쓰는 사람을 위한 루틴 라이브러리와 로그인 없이 바로 쓰는 게스트 모드도 포함.
  - k: "타겟 / Who"
    h: 루틴을 메모장으로 관리하던 사람
    p: 제작자 본인이 매일 쓰려고 만든 개인용 도구. 메모장 루틴과 운동 앱 사이를 오가는 불편을 없애는 것이 목적이라, 앱 안 편집 화면도 메모장과 같은 방식으로 설계. 개인용으로 쓰던 앱을 스토어 정책 요건(계정 삭제·개인정보처리방침·연령 확인)까지 갖춰 정식 출시.
  - k: "현황 / Status"
    h: Google Play 정식 출시, 무료·광고 미포함
    p: 2026년 10월 27일 Google Play 정식 출시(대한민국). Kotlin과 Jetpack Compose 기반 네이티브 앱으로, 백그라운드 타이머 정확도 때문에 웹뷰 방식 대신 네이티브 채택. 기록은 오프라인 우선 저장 후 Firebase에 동기화. 무료, 광고 미포함.
play:
  - { label: "Google Play", url: "https://play.google.com/store/apps/details?id=com.saso.routinelog" }
links:
  - { label: "개인정보처리방침", url: "/routinememo/privacy/" }
---
