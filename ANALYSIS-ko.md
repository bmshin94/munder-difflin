# Munder Difflin 분석 정리 (한국어)

> 이 저장소를 처음부터 뜯어보며 나눈 대화를 정리한 문서입니다.
> 작성일: 2026-09-01 / 기준 버전: v0.4.6 (pre-release)

## 🔗 저장소 주소

| 구분 | 주소 |
|---|---|
| **원본 (upstream)** | https://github.com/chaitanyagiri/munder-difflin |
| **내 포크 (origin)** | https://github.com/bmshin94/munder-difflin |
| 공식 홈페이지 | https://munderdiffl.in/ |
| 릴리즈 (설치파일) | https://github.com/chaitanyagiri/munder-difflin/releases/latest |
| Agent Gallery | https://munderdiffl.in/hires/ |
| 블로그 | https://munderdiffl.in/blog/ |
| Discord | https://discord.gg/SEDzP5ZPk5 |

---

## 1. 이게 뭐하는 프로젝트야?

**한 줄 요약: "AI 코딩 CLI를 여러 개 동시에 돌려서, 서로 대화하고 기억하는 'AI 직원 사무실'로 만들어주는 데스크톱 앱"**

이름은 미드 《The Office》의 "Dunder Mifflin(던더 미플린) 제지회사" 패러디입니다.
슬로건도 *"세계 최고의 에이전트, 세계 최악의 제지회사"*.

| 항목 | 내용 |
|---|---|
| 버전 / 상태 | v0.4.6 · **pre-release (정식 아님)** |
| 라이선스 | MIT (무료, 상업적 이용 가능) |
| 기술 스택 | Electron · React · TypeScript · Pixi.js · xterm.js · node-pty |
| 규모 | TS/TSX 약 **6.4만 줄**, 전체 1,884 파일 |
| 성과 | GitHub Trending **1위**, Product Hunt 일간 **5위** |

### 핵심 특징 4가지

1. **터미널 = 진짜 AI 직원**
   `node-pty`로 **실제 CLI 프로세스**를 띄웁니다. 가짜 API 호출이 아니라 터미널에서 치는 그 `claude` 그대로라서,
   **이미 결제한 구독(Claude Pro/Max 등)의 시간당 한도**를 그대로 씁니다. 추가 API 비용 없음.

   지원 엔진 12종: Claude Code · Codex(GPT) · Gemini CLI · Antigravity · Grok · Kimi ·
   Qwen · OpenCode · Crush · Pi · Copilot CLI · Cursor (+ 커스텀, + Ollama/LM Studio 로컬 LLM)

2. **AI 직원들이 도트 그래픽 사무실을 걸어다님**
   Pixi.js 2D 오피스 맵에서 아바타가 자기 자리로 걸어가고, 서로 메시지 보내면 편지봉투가 날아다닙니다.
   디자인 컨셉은 *"동물의 숲 × 마더2 × SNES 메뉴 UI"*.

3. **'마이클'이라는 팀장 AI (GOD 에이전트)**
   사용자는 마이클한테만 말하면 됩니다. 마이클이 일감 배분, 에이전트 간 중재를 하고,
   **돈 쓰는 일 / 파일 지우는 일 / 범위 변경만** 사람에게 승인 요청합니다.

4. **Hive(벌집) — 기억하고 소통하는 구조**

   ```
   hive/
     registry.json     ← 직원 명부 (역할, 능력, 자리)
     board.md          ← 공용 칠판 (다 같이 보는 계획서)
     tasks.json        ← 업무 대장
     log.jsonl         ← 모든 사건 기록
     agents/<id>/
       identity.md     ← "나는 누구인가" (시작할 때 읽음)
       memory.md       ← 장기 기억 (배운 거 계속 추가)
       inbox/          ← 나한테 온 메시지함
   ```

   **똑똑한 설계 포인트:** git 커밋은 **Electron 메인 프로세스 하나만** 합니다.
   에이전트 여러 개가 동시에 git을 건드리면 `.git/index.lock`이 깨지기 때문에,
   에이전트는 파일만 쓰고 배달은 라우터가 대신합니다. (single-committer 설계)

---

## 2. 폴더 구조

| 폴더/파일 | 역할 |
|---|---|
| `src/main/` (49개) | **핵심 두뇌.** `hive.ts`(3,303줄), `pty.ts`(터미널), `breaker.ts`(폭주 차단기), `telemetry.ts`, `slack.ts`, `webhook.ts` |
| `src/renderer/` | React UI. `scene/office/`가 도트 사무실, `components/`에 칸반·메모리그래프·Monaco IDE |
| `src/preload/` | 보안 다리. 렌더러는 `window.cth`로만 시스템 접근 |
| `src/shared/` | 양쪽 공용 타입/카탈로그 |
| `docs/` (841개) | 랜딩페이지(munderdiffl.in) + 로고 + 스크린샷 |
| `blog/` (576개) | 공식 블로그 글 |
| `test/` (98개) | 테스트 |
| `landing-remotion/` | 홍보 영상 렌더링 프로젝트 |
| **문서 6종** | `HIVE.md`(멀티에이전트) · `SPEC.md`(터미널) · `DESIGN.md`(비주얼) · `MEMORY_GRAPH_SPEC.md` · `TELEMETRY.md` · `CHANGELOG.md` |

> 참고: 루트의 `index-j0JdoH0M.js` (12MB)는 원작자가 실수로 커밋한 Pixi.js 빌드 결과물입니다. 소스가 아닙니다.

---

## 3. 설치 방법

### 🅰️ 쉬운 길 — 완성된 앱 다운로드 (추천)

https://github.com/chaitanyagiri/munder-difflin/releases/latest

macOS(서명+공증 완료) / Windows / Linux 빌드 제공. 그냥 써볼 거면 이쪽.

### 🅱️ 개발자 길 — 소스 빌드

**준비물**

| 필요한 것 | 확인 | 없으면 |
|---|---|---|
| Node.js 18+ | `node -v` | https://nodejs.org |
| C/C++ 컴파일러 | — | 아래 참고 |
| AI CLI 최소 1개 | `claude --version` | Claude Code 등 설치 |

```bash
# 컴파일러 설치 (node-pty가 네이티브 애드온이라 필수)
xcode-select --install                          # macOS
npm install --global windows-build-tools        # Windows (관리자 PowerShell)
sudo apt install build-essential python3        # Ubuntu/Linux
```

**실행**

```bash
git clone https://github.com/bmshin94/munder-difflin.git
cd munder-difflin
npm install     # 2~5분. node-pty를 Electron ABI에 맞춰 재빌드
npm run dev     # 앱 실행
```

**기타 스크립트**

```bash
npm run build         # 배포용 빌드
npm run typecheck     # 타입 검사
npm run test:focused  # 테스트
npm run dist:mac      # 설치파일 생성 (dist:win / dist:linux)
```

---

## 4. 사용법

### 첫 실행 — 온보딩 마법사 7단계

| 단계 | 내용 |
|---|---|
| 1. Persona | **기술자 / 비기술자** 선택 → 설명 난이도가 바뀜 |
| 2. Welcome | 기능 소개 |
| 3. **Harness Home** ⭐ | AI 기억·메시지 저장 폴더 (기본 `~/HarnessAgents`). **지우면 기억상실!** |
| 4. **Orchestrator** ⭐ | 팀장(마이클) 엔진 선택. 설치 여부 자동 검사 + 자동 설치 지원 |
| 5. Repos | AI가 작업할 실제 프로젝트 폴더 등록 |
| 6. **Permissions** ⭐ | **Auto mode**(자동승인, 기본 ON — 처음엔 끄는 걸 추천) / 익명 통계 전송 |
| 7. Done | 사무실 화면 진입 |

### 직원 뽑기 — Add Agent (4개 탭)

| 탭 | 항목 |
|---|---|
| **Identity** | 이름(기본 `Jim`) / 캐릭터 15종 / 색상 |
| **Workspace** | 작업 폴더 / **Isolate**(자기만의 git worktree — 여러 명 굴릴 땐 필수!) / Resume 세션ID |
| **Engine** | 엔진 + 모델 선택 (직원마다 다르게 가능) |
| **Briefing** ⭐ | Description(역할) / **Goal(계속 추구할 목표)** — 여기가 핵심 |

**템플릿 5종:** Repo Janitor · Docs Writer · Bug Triager · Research Assistant · Release Manager

### Command Center 탭 11개

| 탭 | 역할 |
|---|---|
| Terminal | 마이클과 직접 대화 (여기서 시작) |
| Floor | 사무실 전체 현황 |
| Tasks | 칸반보드, 업무 의존관계 관리 |
| ASK ME | AI가 사람에게 던지는 질문 |
| Triggers | 자동 실행 (예약 / 슬랙 / 웹훅) |
| Memory · Graph | 기억 검색 / 기억 지도 |
| Activity | 전체 활동 로그 |
| Skills | 227개 스킬 카탈로그 |
| Workers | 임시 직원 관리 |

> **메시지 큐:** AI가 일하는 중이면 입력이 대기열에 쌓였다가 손이 비면 자동 전달됩니다.
> (`docs/message-queue.md` 참고 — 타이핑이 섞이는 걸 막는 장치)
> 급하면 대기열에서 **send now**로 강제 전송 가능.

### Settings 7개 섹션

`General` · `Prerequisites`(설치 현황 한눈에) · `Agents & Models` ·
**`Autonomy & Budgets`**(토큰 예산 / 폭주 차단기: steer → constrain → stop 3단계) ·
`Connections`(슬랙·웹훅·API키) · `Voice`(음성 명령) · `Memory & Knowledge`

### 자주 터지는 문제

| 증상 | 해결 |
|---|---|
| `npm install` 실패 | C/C++ 컴파일러 없음 |
| `node-pty` 로드 실패 | `npm install` 재실행 (Electron ABI 재빌드) |
| 에이전트가 안 켜짐 | Settings → Prerequisites에서 CLI 설치 확인 |
| 여러 AI가 코드 충돌 | Add Agent에서 **Isolate** 켜기 |
| 기억 소실 | Harness Home 폴더 삭제 여부 확인 |

---

## 5. 수익화 분석

### 원작자의 현재 수익 모델

`docs/wall.html` + `docs/wall-data.json` 분석 결과:

| 항목 | 내용 |
|---|---|
| 상품 | **Founders' Wall** (창립 후원자의 벽) |
| 가격 | **$20 (1회성)** |
| 혜택 | 홈페이지 이름 각인 + PRO 연간플랜 50% 할인 + 1개월 무료 |
| 한정 | 선착순 100명 |
| **실적** | **78명** (2026-08-14 ~ 08-31, 약 2.5주, **약 $1,560**) |

**핵심 발견:** `src/` 전체에서 `isPro` / `paywall` / `entitlement` 같은
**유료 기능 게이트 코드가 발견되지 않음** → PRO는 아직 존재하지 않습니다.

> ✅ 수요는 검증됨 (실제로 돈을 냄)
> ⚠️ 제품은 미완성 (유료 기능이 뭐가 될지도 미정)

### 상업화 시 법적 조건 3가지

| 조건 | 내용 |
|---|---|
| **① LimeZu 크레딧 필수** | 도트 타일셋이 유료 에셋(2026-08-20 구매, Complete Version 라이선스). **README·앱 크레딧·웹사이트 3곳에 https://limezu.itch.io/ 링크 유지 필수.** 에셋 자체 재판매는 금지 |
| **② 브랜드 교체 필수** | "Munder Difflin"은 NBC 《The Office》의 던더 미플린 패러디. README에도 *"애정 어린 패러디이며 NBC와 무관"*이라고 명시. **상업화하면 상표권 리스크가 커지므로 이름·로고·캐릭터 전면 교체 필요** |
| **③ MIT 고지 유지** | 원작자 저작권 문구 유지 |

> 참고: 캐릭터 스프라이트는 `portraitArt.ts`에서 절차적으로 그리는 **자체 제작물**이라
> 제3자 라이선스가 없습니다. (LimeZu 에셋은 타일셋 3종뿐)

### 아이디어 랭킹

#### 🥇 1위. AI 코딩 비용 대시보드

**이유: 이 저장소에 이미 다 만들어져 있음**

```
src/main/transcript.ts   ← Claude 대화기록에서 실제 토큰/비용 추출
src/main/pricing.ts      ← 모델별 단가 계산
src/main/costLifetime.ts ← 누적 비용
src/main/db.ts           ← SQLite 영구 원장(ledger)
src/main/telemetry.ts    ← OTel 추적
```

- 파는 말: *"우리 개발팀이 AI에 이번 달 얼마 썼는지, 누가 뭘로 썼는지 한눈에"*
- 회사는 **비용 가시성**에 지갑을 엶 (결재가 잘 남)
- 사무실 앱 전체가 아니라 **대시보드만** 필요 → 진입장벽 낮음
- **도트 캐릭터를 안 쓰므로 상표 리스크 회피**
- 팀 규모 과금 → 구독 모델 자연스러움

#### 🥈 2위. 한국어판 + 설치/운영 대행

- **지원 언어가 영어 / 중국어(간체) / 아랍어뿐 — 한국어 없음**
- 설치 난이도(네이티브 빌드, CLI 12종)가 높아 **대행 자체가 상품**
- 기업 세팅 대행(건당) / 호스팅 서비스(월 구독) / 한국어 번역 오픈소스 기여로 신뢰 확보

#### 🥉 3위. 에이전트 역할 팩 판매

- `docs/hires/manifests`에 **22개뿐** (JSON 하나 = 직원 하나)
- 산업별 묶음 판매: 커머스 팩 / 핀테크 팩 / 게임사 팩
- 장점: 원가 0, 하루면 제작 / 단점: 단가 낮음($5~30)

#### 4위. 기업용 오픈코어

- 현재 없는 것: 로그인/계정, 팀 권한, 중앙 정책관리, 감사 로그
- 매출 규모는 가장 크지만 **원작자 PRO와 정면충돌**, 개발 6개월+

#### 5위. 콘텐츠 & 교육

- GitHub 트렌딩 1위 = **검색 수요 폭발 중**, 한국어 콘텐츠 없음
- 유튜브 / 블로그 / 강의 → **현금화 최단거리, 리스크 0**

### 추천 로드맵

```
[1개월]   5번 콘텐츠 시작        → 비용 0, 시장 반응 확인, 초기 고객 확보
   ↓
[2~3개월] 2번 한국어 번역 기여   → 원작자와 우호 관계 + "한국 담당자" 포지션 선점
   ↓
[3개월~]  1번 비용 대시보드      → 기존 코드 재활용, 브랜드 리스크 0, 구독 수익
```

- 처음부터 4번(기업용)은 위험 — 6개월 개발 후 원작자 PRO가 나오면 무의미해짐
- 1번은 원작자와 **경쟁이 아니라 보완**. Claude Code만 쓰는 사람도 고객이 됨

---

## 6. PHP로 만들 수 있나?

### ❌ 앱 전체 클론 → 비추천

막히는 지점은 딱 하나, **PTY(의사 터미널)** 입니다.

```typescript
// src/main/pty.ts:1
import * as pty from 'node-pty';
```

| 문제 | 설명 |
|---|---|
| PHP에 PTY가 없음 | `proc_open()`은 **파이프**만 제공하고 TTY가 아님. 대화형 CLI는 `posix_isatty()`가 false면 대화형 UI를 띄우지 않음 |
| FFI 우회의 비용 | PHP 7.4+ FFI로 libutil `forkpty()` 호출은 이론상 가능하나 창 크기 조절·시그널·Windows 지원을 전부 직접 구현해야 함 |
| 실행 모델 불일치 | PHP-FPM은 요청-응답 모델. 여기선 몇 시간 살아있는 프로세스를 계속 붙들어야 함 |
| 애초에 데스크톱 앱 | Electron + Pixi.js 기반 |

> 하이브리드(PHP 웹 + Node 사이드카)는 가능하지만 관리 포인트가 2배가 됩니다.

### ✅ 1위 아이디어(비용 대시보드)는 PHP로 최적

이 기능엔 **PTY가 전혀 필요 없습니다.** 하는 일은:

```
JSONL 파일 읽기 → 숫자 합산 → 단가 곱하기 → 웹 화면 출력
```

**① 파일 위치 규칙** (`transcript.ts` 기준)

```
~/.claude/projects/{키}/{세션ID}.jsonl
키 = 작업폴더 절대경로에서 영숫자가 아닌 모든 문자를 '-' 로 치환
예) /Users/me/app → -Users-me-app
```

**② 필요한 JSONL 필드 4개**

```
message.usage.input_tokens
message.usage.output_tokens
message.usage.cache_creation_input_tokens
message.usage.cache_read_input_tokens
```

**③ 단가표** (`pricing.ts`, 100만 토큰당 USD)

| 모델 | 입력 | 출력 | 캐시읽기 | 캐시쓰기 |
|---|---|---|---|---|
| Opus | 15 | 75 | 1.5 | 18.75 |
| Sonnet | 3 | 15 | 0.3 | 3.75 |
| Haiku | 0.8 | 4 | 0.08 | 1.0 |

> 모델을 알 수 없으면 Sonnet으로 가정하는 것이 원본 동작입니다.

**④ PHP 포팅 예시**

```php
<?php
// 폴더 경로 → Claude 프로젝트 키
function projectKey(string $cwd): string {
    return preg_replace('/[^a-zA-Z0-9]/', '-', $cwd);
}

const PRICES = [
    'opus'   => ['in' => 15.0, 'out' => 75.0, 'cRead' => 1.5,  'cWrite' => 18.75],
    'sonnet' => ['in' => 3.0,  'out' => 15.0, 'cRead' => 0.3,  'cWrite' => 3.75],
    'haiku'  => ['in' => 0.8,  'out' => 4.0,  'cRead' => 0.08, 'cWrite' => 1.0],
];

function priceFor(?string $model): array {
    $m = strtolower($model ?? '');
    foreach (['opus', 'haiku', 'sonnet'] as $family) {
        if (str_contains($m, $family)) return PRICES[$family];
    }
    return PRICES['sonnet'];
}

// JSONL 스트리밍 파싱 — 파일이 커도 메모리 안전
function sumUsage(string $jsonlPath): array {
    $total = ['in' => 0, 'out' => 0, 'cRead' => 0, 'cWrite' => 0, 'usd' => 0.0];
    $fh = fopen($jsonlPath, 'r');
    if (!$fh) return $total;

    while (($line = fgets($fh)) !== false) {
        $rec = json_decode(trim($line), true);
        $u = $rec['message']['usage'] ?? null;
        if (!$u) continue;

        $p      = priceFor($rec['message']['model'] ?? null);
        $in     = (int)($u['input_tokens'] ?? 0);
        $out    = (int)($u['output_tokens'] ?? 0);
        $cWrite = (int)($u['cache_creation_input_tokens'] ?? 0);
        $cRead  = (int)($u['cache_read_input_tokens'] ?? 0);

        $total['in']     += $in;
        $total['out']    += $out;
        $total['cRead']  += $cRead;
        $total['cWrite'] += $cWrite;
        $total['usd']    += ($in * $p['in'] + $out * $p['out']
                          +  $cRead * $p['cRead'] + $cWrite * $p['cWrite']) / 1000000;
    }
    fclose($fh);
    return $total;
}
```

**⑤ 추천 스택**

```
Laravel + MySQL + Chart.js
  ├─ 수집: 각 PC의 작은 에이전트가 JSONL을 읽어 API로 전송
  ├─ 저장: usage_records 테이블 (파일ID + 읽은 오프셋으로 중복 방지)
  ├─ 집계: 스케줄러로 일/주/월 롤업
  └─ 화면: 팀별·사람별·모델별 비용 대시보드 + 예산 알림
```

> 원본은 `usageCache`로 **파일 크기를 비교해 새로 늘어난 부분만 파싱**합니다.
> PHP에서도 `filesize()` + 마지막 오프셋을 저장하면 동일하게 구현됩니다.

### 정리

| 만들 것 | PHP 가능? |
|---|---|
| Munder Difflin 클론 전체 | ❌ Node/Electron 필요 |
| **AI 비용 대시보드 (1위)** | ✅ **최적. Laravel이 오히려 빠름** |
| 에이전트 역할 팩 마켓 (3위) | ✅ 일반 웹서비스라 문제 없음 |
| 한국어 번역 기여 (2위) | ✅ JSON 파일이라 언어 무관 |

---

## 7. 주의사항 요약

1. **아직 pre-release** (v0.4.6). 중요한 프로덕션 코드베이스에 바로 붙이는 것은 권장하지 않음
2. **최소 1개의 AI CLI 구독 필요** — 앱은 무료지만 Claude Code 등은 별도 결제
3. **익명 텔레메트리 전송** — `TELEMETRY.md`에 전체 목록 명시(앱 버전·OS·실행 횟수 등, 메시지 내용은 미전송). 온보딩 6단계에서 끌 수 있음
4. **네이티브 빌드 필요** — `node-pty` 때문에 C/C++ 툴체인 필수
5. 루트 `index-j0JdoH0M.js` (12MB)는 실수로 커밋된 빌드 산출물
