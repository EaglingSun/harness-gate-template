# AI 리뷰어 게이트 — 3일차 M5 실습 저장소 (템플릿)

PR 을 올리거나 병합하면 **내 PC 의 러너**에서 세 가지가 돈다.

| workflow | 언제 | 하는 일 |
|---|---|---|
| `pr-review` | `src/` 를 바꾼 PR | AI 리뷰어가 diff 를 보고 PR 에 코멘트. high 1건 이상이거나 리뷰가 실패하면 FAIL |
| `harness-gate` | 리뷰어(`harness/`) · 정답셋(`evals/`) · 결과(`results/`)를 바꾼 PR | 지금 리뷰어 버전(해시)의 측정 결과가 커밋돼 있고 기준을 넘을 때만 PASS |
| `deploy` | `main` 에 들어온 것(PR 병합) | gcc 로 `src/compute.c` 빌드 → `tests/test_compute.c` 시험 → `libcompute.a` · `compute.h` 를 Actions 아티팩트로(= 이 실습의 배포). gcc 는 S0 에서 받은 `%PUBLIC%\vsm-gcc` |

들어 있는 리뷰어 버전과 그 측정 결과는 `py kit.py hash` 와 `results/` 폴더에서 본다.

## 1. 내 저장소 만들기

이 페이지 위 **Use this template → Create a new repository** → 이름 `harness-gate` · **Private**.

## 2. 받기

PowerShell:
```
cd C:\lab\day3
git clone https://github.com/<내 계정>/harness-gate.git
```
처음이면 브라우저 로그인 창이 뜬다.

## 3. 러너 연결

내 저장소 → **Settings → Actions → Runners → New self-hosted runner → Windows · x64**.
새 PowerShell 창에서 `cd C:\lab\day3` 뒤, 화면의 Download · Configure 명령을 첫 줄(`mkdir actions-runner; cd actions-runner`)부터 차례로 붙여 넣는다(질문은 모두 Enter). 마지막 `./run.cmd` 창은 **수업이 끝날 때까지 열어 둔다** — `Listening for Jobs` 가 보이면 연결된 것이다.
job 이 `Queued` 에서 멈추면 러너 창이 열려 있는지 본다. 러너를 껐다 켠 직후 2 ~ 3분은 `A session for this runner already exists` 가 날 수 있다 → Cancel 뒤 다시 Re-run.

## 4. PR 흐름

`C:\lab\day3\harness-gate` 에서 연 Claude Code 에 시킨다. PR 은 push 뒤 GitHub 화면의 **Compare & pull request** 로 만든다.

1. **코드 PR** — 「브랜치 m5-bug 를 만들고 src/compute.c 의 average() 에서 count 를 확인하는 가드를 지운 뒤 커밋하고 push 해 줘」
   → `pr-review` 가 돈다 → PR 에 AI 리뷰어 코멘트 · 체크 FAIL
2. **고쳐서 병합** — 「브랜치 m5-bug 의 average() 가드를 values 가 NULL 이거나 count 가 0 이하면 0 을 돌려주게 고쳐 커밋하고 push 해 줘」
   → `pr-review` 가 다시 돈다 → PASS 면 Merge → `deploy` 가 빌드 · 시험 · 아티팩트(`compute-<번호>`)
   (main 과 똑같이 되돌리면 PR 의 바뀐 파일이 0 이 되어 리뷰가 돌지 않는다)
3. **리뷰어 PR** — 「main 에서 브랜치 m5-rule 을 만들고 harness/conventions.md 에 규칙 한 줄을 더해 커밋하고 push 해 줘」
   → `harness-gate` 가 돈다 → 측정 결과가 없어 FAIL

## 수업 뒤 정리 (선택)

러너 창에서 Ctrl+C → 저장소 Settings → Actions → Runners 에서 내 러너 → **Remove** (또는 `cd C:\lab\day3\actions-runner` → `./config.cmd remove` · 토큰은 같은 화면).
