---
layout: post
title: "무인 엔진 게임 개발 실전"
description: "이터널 리턴의 턴제 전투 팬게임을 무인 작업 엔진에 맡긴 5일의 기록. 셸 없는 작업자의 밸런스 측정 문제, 80초에서 570초로 늘어난 테스트 시간, 시험 하나에 2시간 44분 멈춘 기계검사까지."
date: 2026-10-21 00:40:00 +0900
tags: [build-in-public, ai-agents, godot, game-dev]
---

*🇬🇧 [English version below](#english-version)*

[작업 엔진 v6 회고](/2026/10/15/agent-engine-v6-retrospective/)와 [무인 엔진 관측과 정지](/2026/10/15/unattended-engine-observe-and-stop/)의 주제는 엔진 자체. 이 글의 주제는 그 엔진에 실제 게임 개발을 맡긴 기록.

대상은 이터널 리턴의 턴제 전투 팬게임(비공식, 미출시). 실험체가 등장하는 턴제 전투에 스토리를 얹는 싱글 게임이고, 스택은 Godot 4.7.2 + GDScript, 테스트는 GUT. 10월 6일부터 10일까지 전투 코어, 밸런스 시뮬레이터, 전투 화면, 스토리 1부를 엔진의 무인 실행(run)으로 진행.

| 역할 | 담당 |
|---|---|
| Master (Claude Code 세션) | 할 일 목록 작성, 끝 승인, 엔진 밖 측정 대행 |
| 작업자 AI | 코드·데이터 수정 |
| 평가자 AI | 결과 판정, 시나리오 실행 |
| 사람 | 방향 결정, 플레이 테스트, 규칙 개편 뒤 수치 확정 |

기록할 장면은 셋. 셋 모두 「작업자가 직접 돌릴 수 없는 것」과 연결된 문제.

## 0:1000 상성과 셸 없는 작업자

밸런스 측정 도구는 헤드리스 시뮬레이터. 같은 편성끼리 1000판, 1:1 조합마다 1000판 같은 방식. 걸림돌은 작업자 AI에게 셸이 없다는 점. 시뮬레이터는 작업자가 질문으로 요청하면 Master가 대신 돌려 결과를 답하는 구조.

첫 단계(실험체 3명)에서 Master 측정은 3회, 조정은 3차. 2차 조정 뒤 측정에서 한 조합의 1:1 결과가 0:1000. 한쪽의 1000판 전패에 평균 누적 피해 0.8. 3차 조정은 재측정 상한에 걸려 작업 단계에서는 미측정. 끝 승인 때 Master가 재현 측정한 결과도 여전히 0:1000.

원인 서술의 경과도 기록 대상. 작업자의 첫 밸런스 문서에는 「원작 규칙대로의 현상」이라는 서술이 존재. 평가자가 이 절을 반려했고, 작업자는 그 서술을 근거 없는 단정으로 철회한 뒤 코드로 확인한 사실과 미측정 가설을 분리. 다만 셸이 없으니 어느 가설이 맞는지는 판정 불가.

판정은 Master가 진단 테스트를 직접 실행해 완료. 시드 5개에서 약한 쪽이 낸 공격 코인 59개의 행방은 아래 표.

| 코인 행방 | 개수 |
|---|---:|
| 적중 | 0 |
| 회피에 빗나감 | 0 |
| 합(서로를 노린 공격의 맞대결)에서 패배 | 59 |

서로를 노린 공격은 합으로 처리하고, 위력 합이 큰 쪽이 이기며 진 쪽은 코인을 잃는 규칙. 상대는 코인이 많은 연사 스킬에 속도까지 빨라 합을 거는 쪽이었고, 약한 쪽은 앞선 하향 조정으로 위력이 낮아진 상태. 결론은 구현 버그가 아닌 합 위력 격차.

## 측정을 작업자 손으로

다음 단계(나머지 4명 추가, 7명 밸런스)에서 바꾼 것은 엔진 코드가 아닌 실행 설정. 엔진에는 자기 검사(self_check) 장치가 이미 존재. 작업자에게 허용된 유일한 셸 명령이고, 지정한 검사를 돌려 출력을 작업자에게 반환하는 구조. 엔진도 같은 검사를 다시 돌려 자기 결과만 신뢰. 여기에 밸런스 측정 스크립트를 연결.

```json
"self_check": {
  "commands": ["godot --headless --path . -s sim/balance_check.gd -- {files}"],
  "extensions": [".json"],
  "timeout_sec": 600
}
```

작업자가 수치 파일(`.json`)을 고친 뒤 자기 검사를 실행하면 측정이 돌고 기준표가 반환되는 구조. 측정 스크립트는 기준 미달이어도 항상 통과(종료 코드 0)로 끝나고 표만 출력. 실패로 막는 장치가 아니라 판단 재료를 건네는 장치라는 설계.

<figure class="diagram"><div class="fig-scroll"><a href="/assets/img/posts/unattended-engine-game-dev/measure-path.ko.svg"><img src="/assets/img/posts/unattended-engine-game-dev/measure-path.ko.svg" width="720" height="330" loading="lazy" alt="이전: 셸 없는 작업자가 측정을 질문으로 요청하고 Master가 시뮬레이터를 돌려 답변, 재측정 상한에 걸려 작업 중 마지막 조정값은 미측정. 이후: 작업자가 수치 파일을 고친 뒤 자기 검사를 직접 실행하면 balance_check가 돌아 기준표 반환, 기준 10개 중 통과가 5개에서 10개로."></a></div><figcaption>엔진 코드 변경 0건. 실행 설정에 자기 검사 항목만 추가.</figcaption></figure>

결과는 아래.

| 시점 | 기준 10개 중 통과 |
|---|---:|
| 4명 추가 직후, 첫 측정 | 5 |
| 작업자가 조정마다 직접 측정한 뒤 | 10 |

기준은 네 묶음: 7명 무작위 편성에서 실험체별 승률, 역할 정체성(탱커 생존 최장, 원거리 딜러 누적 피해 최대 등), 같은 편성끼리 미러전 균형, 1:1 조합 21쌍 중 지원가가 낀 쌍을 뺀 15쌍에서 한쪽 승률 95% 이하. 0:1000이던 조합도 이 기준 안으로 복귀.

통과와 별개로 남은 한계도 작업자가 문서에 명시. 극단 상성 일부는 승률이 아닌 무승부로 해소. 방어가 강한 쪽과 회피가 맞물려 100턴 상한까지 가는 교착이고, 기준은 통과지만 재미 면에서 좋은 해소는 아니라는 평가. 수치 1~2 변경에 다른 지표 순위가 뒤집히는 민감한 상태라는 경고도 함께.

## 테스트 시간 80초 → 570초

전투 화면 단계 뒤 전체 테스트 시간이 80초에서 570초로 증가. 그중 화면 시험 파일 하나가 단독 478초. 다음 run의 요청서에 넣은 조건은 하나. 시험 삭제나 판 수 축소로 숨기는 방식은 금지, 근본 원인 수정.

원인은 화면 갱신 구조. 갱신마다 노드 약 200개를 새로 만들고 이전 노드는 `queue_free`로 해제 예약. 해제는 프레임이 넘어갈 때 처리되는데, 헤드리스 시험에서는 시험 하나가 끝날 때까지 프레임이 진행되지 않는 구조. 한 판에 갱신이 수천 번이니 해제 대기 노드가 계속 누적되고, 그때마다 글자 수백 개의 모양 재계산까지 반복.

<figure class="diagram"><div class="fig-scroll"><a href="/assets/img/posts/unattended-engine-game-dev/node-pool.ko.svg"><img src="/assets/img/posts/unattended-engine-game-dev/node-pool.ko.svg" width="720" height="320" loading="lazy" alt="이전: 화면 갱신마다 노드 약 200개 생성, 이전 노드는 queue_free. 헤드리스 시험에서는 프레임이 진행되지 않아 해제 대기 노드가 시험 끝까지 누적, 전체 테스트 570초. 이후: 칠판·글자·버튼을 풀에서 번호순 재사용, 바뀐 값만 덮어쓰고 남는 것은 숨김 처리, 갱신 뒤 노드 수 불변 시험 추가, 36초."></a></div><figcaption>원인은 코드 분석으로 판정. 기존 시험 삭제 0건.</figcaption></figure>

수정은 노드 풀. 칠판·글자·버튼은 한 번 만든 노드를 번호순으로 재사용하고 바뀐 값만 덮어쓰기, 남는 노드는 숨김 처리. 여기에 「갱신 뒤 노드 수 불변」 시험을 추가해 같은 회귀의 재발을 차단. 결과는 36초.

단서 하나. 이 원인은 작업자의 코드 분석으로 정한 것이고 실측은 미실시. 작업자에게 셸이 없어서였고, 설계 기록에도 그대로 명시.

## 시험 하나에 2시간 44분

전투 규칙 개편 run에서 엔진의 기계검사(전체 테스트 실행)가 2시간 44분간 정지.

원인은 두 겹. 제품 쪽에서는 자동 배치가 빈 슬롯을 남기는 경로가 존재. 앞 슬롯 계획을 바꾸면 뒤 슬롯 계획이 비는 흐름이었고, 이 진단은 작업자의 정적 분석이며 실행 추적으로 확정하지는 못한 상태. 시험 쪽에서는 스토리 진행 시험의 자동 진행 반복이 거절 여부를 확인하지 않는 구조. 거절된 시도를 끝없이 반복하는 반복문이 기계검사 전체를 붙잡은 셈.

엔진 쪽 의문도 하나. 기계검사 명령에는 기본 30분 상한이 설정된 상태였는데도 2시간 44분간 미종료. 상한이 작동하지 않은 원인은 아직 미확인.

| 층 | 조치 |
|---|---|
| 시험 | 거절 즉시 사유 출력 후 실패, 반복 상한 추가 |
| 제품 | 자동 배치를 빈 슬롯이 없어질 때까지 여러 바퀴 반복 |
| 요청서 | 이후 모든 run에 「반복문은 거절·상한 확인, 무한 반복 금지」 조건 |
| 엔진 | 기계검사 시간 제한 문제를 엔진 쪽 과제로 등록 |

수정 과정에도 걸림이 하나. 문제의 시험 파일이 할 일 목록의 수정 대상 밖이라 작업자가 수정 여부를 질문했고, 질문 횟수가 상한(3회)을 넘어 엔진이 사람 도움 요청으로 정지. Master가 그 파일을 후속 항목으로 추가해 재개.

## 정리

| 장면 | 막힌 지점 | 조치 |
|---|---|---|
| 0:1000 상성 | 측정이 Master 대행이라 질문 왕복과 상한 | 자기 검사에 측정 연결 |
| 570초 회귀 | 원인 실측 불가 | 코드 분석 + 노드 수 불변 시험 |
| 2시간 44분 정지 | 무한 반복 시험, 기계검사 상한(30분) 미작동 | 시험 상한, 요청서 조건, 엔진 과제 |

- 무인 실행에서 작업자가 직접 돌릴 수 없는 측정은 병목. 질문 왕복보다 자기 검사 연결이 효과적.
- 기준 통과와 좋은 결과는 별개. 교착 무승부로 통과한 상성처럼, 통과 뒤의 한계를 숨기지 않는 기록이 필요.
- 기다리는 시간에도 상한 필요. 그리고 상한은 설정 여부가 아니라 실제로 끊기는지까지 확인 대상. 30분 상한이 있던 명령이 2시간 44분을 버틴 사례.

---

## English version

[The v6 retrospective](/2026/10/15/agent-engine-v6-retrospective/) and [Watching and stopping an unattended engine](/2026/10/15/unattended-engine-observe-and-stop/) were about the engine itself. This one is about what happened when I handed it a real game to build.

The game is an unofficial, unreleased turn-based combat fan game based on Eternal Return: single-player, the game's characters fighting turn-based battles, with a story on top. It's built in Godot 4.7.2 with GDScript and tested with GUT. From October 6 to 10, the combat core, a balance simulator, the battle screen and the first story arc were all built through the engine's unattended runs.

| Role | Who |
|---|---|
| Master (a Claude Code session) | writes the task list, gives final approval, runs measurements the engine can't |
| Builder AI | edits code and data |
| Evaluator AI | judges results, runs scenarios |
| Me | sets direction, playtests, signs off on the numbers after the rules rework |

Three moments are worth writing down, and all of them come back to the same thing: something the builder couldn't run on its own.

### A 0:1000 matchup and a builder with no shell

Balance is measured with a headless simulator: 1,000 games of a lineup against itself, 1,000 games per one-on-one pairing, and so on. The catch is that the builder AI has no shell. To get a simulation, it had to ask a question, and the Master would run it and reply with the numbers.

The first stage, with three characters, got three measurements and three rounds of tuning. After the second round, one one-on-one pairing came back 0:1000. One side lost every game and averaged 0.8 total damage. The third round hit the re-measure cap and went unmeasured during the build; when the Master re-ran it at final approval, the pairing was still 0:1000.

How the cause got written up is worth noting too. The builder's first balance doc explained it as "just how the original rules work." The evaluator rejected that section, and the builder withdrew the claim as unsupported, splitting what it had confirmed in code from hypotheses it hadn't measured. Without a shell, though, it couldn't tell which hypothesis was right.

The Master settled it by running a diagnostic test directly. Across five seeds, here is where the weaker side's 59 attack coins went:

| Where the coins went | Count |
|---|---:|
| Hit | 0 |
| Missed to an evade | 0 |
| Lost in a clash | 59 |

When two attacks target each other they clash: the higher total power wins and the loser starts losing coins. The opponent had a multi-coin rapid-fire skill and enough speed to force clashes; the weaker side had just been tuned down. So it wasn't a bug. It was a raw power gap in clashes.

### Putting measurement in the builder's hands

For the next stage (four more characters, balance across all seven), the change wasn't to engine code. It was to the run config. The engine already had a self-check: the one shell command a builder is allowed to run. It runs a configured check and hands the output back, and the engine later re-runs the same check and trusts only its own result. The Master pointed it at the balance script.

```json
"self_check": {
  "commands": ["godot --headless --path . -s sim/balance_check.gd -- {files}"],
  "extensions": [".json"],
  "timeout_sec": 600
}
```

After editing a stats file (`.json`), the builder runs the self-check and gets back a table of criteria. The script always exits 0, even when criteria fail; it only prints the table. It isn't there to block anything, only to give the builder something to decide with.

<figure class="diagram"><div class="fig-scroll"><a href="/assets/img/posts/unattended-engine-game-dev/measure-path.en.svg"><img src="/assets/img/posts/unattended-engine-game-dev/measure-path.en.svg" width="720" height="330" loading="lazy" alt="Before: the builder, with no shell, asks for a run; the Master runs the simulator and answers; the re-measure cap is hit and the last tuning goes unmeasured mid-run. After: after editing a stats file the builder runs the self-check, which runs balance_check and returns the table; criteria passed go from 5 to 10 of 10."></a></div><figcaption>No engine code changed. Just a self-check entry added to the run config.</figcaption></figure>

| When | Criteria passed (of 10) |
|---|---:|
| First measurement after adding four characters | 5 |
| After the builder measured every tweak itself | 10 |

The criteria fall into four groups: per-character win rate in random seven-character lineups; role identity (the tank survives longest, the ranged dealer deals the most total damage, and so on); balance in mirror matches; and no one-on-one pairing (21 in all, excluding those with the support character) won more than 95% of the time. The 0:1000 pairing came back inside those limits.

The builder also wrote down what passing didn't fix. Some lopsided matchups were resolved by draws, not wins: a strong defender and evasion lock each other up until the 100-turn cap. That passes the criteria but, as the builder put it, isn't a good fix for fun. It also warned that the numbers are tightly coupled; nudging one value by 1 or 2 can flip other rankings.

### Test time: 80 seconds to 570

After the battle-screen stage, the full test suite went from 80 seconds to 570, and one screen test file accounted for 478 of them on its own. The next run's request had one condition: find the root cause, and don't hide it by deleting tests or cutting the number of games.

The cause was how the screen updated. Every update built about 200 new nodes and queued the old ones with `queue_free`. Freed nodes are only cleaned up when a frame advances, and in a headless test no frames advance until that test finishes. With thousands of updates per game, pending nodes piled up, and hundreds of labels had their glyphs laid out again on every update.

<figure class="diagram"><div class="fig-scroll"><a href="/assets/img/posts/unattended-engine-game-dev/node-pool.en.svg"><img src="/assets/img/posts/unattended-engine-game-dev/node-pool.en.svg" width="720" height="320" loading="lazy" alt="Before: each screen update creates about 200 nodes and queue_frees the old ones; in headless tests no frames advance, so freed nodes pile up until the test ends; full suite 570 seconds. After: panels, labels and buttons are reused from a pool in order, only changed values are overwritten, the rest are hidden, and a new test checks that node count stays the same after updates; 36 seconds."></a></div><figcaption>Cause found by code analysis. No existing test was deleted.</figcaption></figure>

The fix was a node pool: panels, labels and buttons are created once and reused in order, only changed values get overwritten, and leftovers are hidden. A new test checks that the node count doesn't grow after updates, so the same regression can't sneak back. The suite came back down to 36 seconds.

One caveat: the cause came from the builder's code analysis and was never measured, again because the builder had no shell. The design log says so.

### 2 hours 44 minutes on a single test

During the run that reworked the combat rules, the engine's mechanical check (the full test run) sat stuck for 2 hours 44 minutes.

There were two layers to it. In the product, auto-placement had a path that left a slot empty: changing the plan for an earlier slot could wipe the plan for a later one. That came from the builder's static analysis and was never confirmed by tracing an actual run. In the tests, the story-flow test's auto-advance loop never checked whether its attempt had been rejected. So it kept retrying a rejected move forever, and that one loop held up the whole check.

There's an open question on the engine side too. The mechanical check had a default 30-minute limit, yet this one ran for 2 hours 44 minutes without being cut off. Why the limit didn't fire is still unknown.

| Layer | Fix |
|---|---|
| Test | fail immediately on rejection with the reason, plus a retry cap |
| Product | auto-placement loops until no slot is left empty |
| Requests | every later run includes "loops must check for rejection and have a cap; no infinite loops" |
| Engine | the mechanical-check time limit issue, filed as an engine task |

Even the fix hit a snag. The test file wasn't on the task list's list of files to edit, so the builder asked whether it could touch it. That pushed it past the question cap (three), and the engine stopped to ask for a human. The Master added the file as a follow-up item and the run resumed.

### Summary

| Moment | What blocked it | What changed |
|---|---|---|
| 0:1000 matchup | measurement went through the Master: question round-trips, then a cap | measurement wired into the self-check |
| 570 s regression | the cause couldn't be measured | code analysis plus a node-count test |
| 2 h 44 min stall | an endless test loop; the 30-minute check limit never fired | retry caps, a request rule, an engine task |

- In an unattended run, any measurement the builder can't run itself becomes the bottleneck. Wiring it into the self-check beats a question round-trip.
- Passing the criteria isn't the same as a good result. When a pass comes with limits, like matchups "fixed" by stalemate draws, write the limits down.
- Waiting needs a cap, and the cap needs checking too: not whether it's configured, but whether it actually cuts things off. This command had a 30-minute limit and still ran for 2 hours 44 minutes.
