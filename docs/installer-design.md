---
title: "설치기 재설계 — 원격 취득 부트스트랩과 로컬 웹 UI"
aliases:
  - "Installer Remote Fetch Bootstrap"
doc_type: "architecture"
scope: "project"
target: "terra-setup"
status: "draft"
version: "v0.1"
last_updated: "2026-10-06"
---

# 설치기 재설계 — 원격 취득 부트스트랩과 로컬 웹 UI

> [!NOTE] 상태
> 2026-10-06 사용자 결정 4건을 담은 초안이다. 구현 전이며, 스키마·키 보관·CI는 §9의 결정이 끝나야 확정된다.

## 1. 왜 다시 만드나

Terra가 git 저장소 셋으로 갈라지면서 설치기가 전제하던 것이 바뀌었다.

| 저장소 | 공개 | 설치기에 닿는 것 |
| --- | --- | --- |
| `StellaxiaLab/Terra` | PRIVATE | 코어 바이너리, 설치기, 코어 번들 |
| `StellaxiaLab/modules` | PUBLIC | 모듈 `.tmod`(릴리스 태그 단위). `lab.stellaxia.node-gui`는 maingui를 따라가는 모듈이다 |
| `StellaxiaLab/maingui` | PRIVATE | 메인 GUI 소스. 사용자에게는 `modules`의 `.tmod`로 간다 |
| `StellaxiaLab/terra-releases` | **PUBLIC (신설)** | 설치기, 코어 아카이브, 서명된 카탈로그 — 소스 없음 |

바뀐 전제:

- 모듈은 Terra 안에서 빌드되지 않는다. `bundled-modules.json`이 modules 릴리스 태그를 핀하고 `build-release`가 `.tmod`를 받아 펼친다.
- 버전 축이 둘이다(코어 `0.5.x`, modules `vYYYY.MM.DD`). 설치 뒤 모듈은 Tree 레지스트리 경로로 갱신된다.
- 설치기는 코어 번들 안에 들어 있어 설치기만 따로 받을 수 없고, Terra가 비공개라 사용자가 익명으로 받을 수도 없다.
- Windows 마법사(PowerShell WinForms 약 1200줄)와 Linux 대화형 마법사가 같은 Go 엔진 위에서 따로 논다.

## 2. 결정 (2026-10-06)

| # | 결정 | 근거 |
| --- | --- | --- |
| D-1 | 모듈(`node-gui` 포함)을 **설치 시점에 원격에서 받는다.** 번들에 동봉하지 않는다 | 모듈은 어차피 `.tmod`로 배포되고 modules가 공개다 |
| D-2 | 프런트는 **로컬 웹 UI**(엔진이 loopback에 서빙) | Windows·Linux 단일 화면, 헤드리스는 SSH 터널 |
| D-3 | 배포는 **공개 릴리스 전용 저장소 `terra-releases`**(권고안 A) | 소스를 공개하지 않고 익명 다운로드를 얻는다 |
| D-4 | 신뢰 루트는 **Terra 서명 키로 서명한 카탈로그**다. GitHub가 아니다 | 저장소·호스트가 뚫려도 카탈로그를 위조할 수 없어야 한다 |

## 3. 구조

```text
사용자 ── terra-setup(부트스트랩, 수 MB, 서명) 하나만 받는다
            │
            ├─ ① release-index.json + .sig 조회   (terra-releases)
            ├─ ② 서명 검증 (설치기에 박힌 공개키)
            ├─ ③ 코어 아카이브 받기 → sha256 검증 → 전개
            ├─ ④ 모듈 .tmod 받기      (modules 릴리스) → sha256 + .tmod 무결성 검증 → 배치
            └─ ⑤ 기존 설치 엔진(setup_engine)이 서비스·설정·등록을 마무리
```

코어와 모듈이 같은 모양이다 — "해시가 박힌 카탈로그 → 다운로드 → 검증 → 배치". 현재 번들 방식에는 카탈로그가 없다.

**변형:** 풀 번들(기존 아카이브 + 모듈 동봉)을 에어갭용으로 계속 낸다. 설치기는 카탈로그의 `url`이 `file:` 또는 로컬 디렉터리(`--source <dir>`)여도 같은 검증 경로를 탄다.

### 3.1 카탈로그 스키마 v3 (초안)

현재 v2는 빌드 트리 안의 경로만 가리키고 URL·sha256·크기가 없다. v3:

```jsonc
{
  "schema_version": 3,
  "version": "0.6.0",
  "commit": "…",
  "issued_at": "2026-10-06T00:00:00Z",
  "expires_at": "2026-12-06T00:00:00Z",
  "min_installer_version": "0.6.0",
  "installer": [
    { "target": "windows-amd64", "url": "…", "sha256": "…", "size": 0 }
  ],
  "core": [
    { "product": "terra-leaf", "target": "linux-amd64", "url": "…", "sha256": "…", "size": 0 }
  ],
  "modules": [
    {
      "id": "lab.stellaxia.node-gui",
      "version": "0.4.0",
      "target": "any",
      "origin": "modules@v2026.10.05",
      "url": "…", "sha256": "…", "size": 0,
      "roles": ["leaf", "tree"],
      "required": true
    }
  ]
}
```

- `modules[]`는 `build-release`가 `bundled-modules.json`의 태그를 **풀어서** 해시까지 새긴다. 원천은 modules 릴리스의 `modules.json`이다(Q-8). 설치기는 모듈 id 목록을 갖지 않는다(기존 원칙 유지: 무엇이 필수인지는 번들이 말한다).
- `required`가 `module_scope=required`의 기준이다(통합 설치 마법사 설계 §Windows GUI와 동일 의미).
- 서명은 `release-index.json.sig`(분리 서명). 형식(minisign·cosign·자체 Ed25519)은 §9 Q-2.
- `issued_at`·`expires_at`·`min_installer_version`은 오래된 카탈로그 재생(rollback)과 낡은 설치기 방어용이다.

## 4. 설치기 흐름 (엔진 관점)

1. 감지 — `install-manifest.json` 유무로 설치 / 유지관리 분기
2. 카탈로그 취득·서명 검증·만료 확인 (실패 시 **아무것도 받지 않고** 원인 표시)
3. 선택 해석 — 역할, 모듈 범위, 옵션 → **받을 목록과 총 크기** 확정(요약 화면의 근거)
4. 스테이징 — 사용자 전용 ACL 임시 디렉터리에 받기(이어받기 허용)
5. 검증 — sha256 → `.tmod` 무결성(`integrity.process.<target>`) → 버전 단조 증가
6. 배치 — 기존 `setAsidePreviousFiles` 방식의 원자적 교체
7. 서비스·설정·등록 — 기존 엔진
8. 실패 시 — 코어는 설치하고 모듈은 **대기** 상태로 남긴다. 이후 Tree 레지스트리 또는 `terra module install`로 채운다

## 5. 로컬 웹 UI

`terra-setup --ui`가 엔진 안에서 서버를 연다. 정적 자산은 `go:embed`.

- loopback(`127.0.0.1`/`::1`) 전용, 무작위 포트
- 1회용 토큰 — URL fragment로 받아 쿠키로 교환하고 즉시 폐기
- Host·Origin 검사(DNS rebinding), CORS 닫음, CSRF 토큰, 엄격한 CSP
- 세션 하나만 허용, 종료 시 토큰 폐기
- **권한 분리:** UI 서버는 비승격. 승격이 필요한 단계는 UAC/`sudo`로 띄운 **별도 프로세스**가 좁은 명령 열거형만 받는다. UI가 임의 명령을 승격 쪽에 넘길 수 없다
- 진행 상황은 엔진이 내보내는 이벤트 스트림(SSE)을 UI가 구독한다. 자식 프로세스 stderr 마지막 N줄·종료 코드 번역·비밀 마스킹은 엔진 책임이다(IN-TODO-02)
- 비대화형 경로(`setup_cli`)는 그대로 둔다. 자동화·CI가 이 경로를 쓴다
- Linux 헤드리스는 loopback만 연다. 외부 바인딩 플래그는 만들지 않는다

화면 흐름과 시각 규칙은 GUI 디자인 브리프가 소유한다.

## 6. 보안 작업 목록

### 6.1 신뢰 루트와 서명
절차는 릴리스 서명 키 절차가 소유한다(루트 키 오프라인 + CI 릴리스 키 + 인증서 + 회전·사고 대응).
- [ ] 서명 키 생성·보관·회전·폐기 절차 — 문서 작성 완료, **키 생성은 사용자가 한다**
- [ ] 설치기에 공개키 박기 + 키 회전용 다중 키 슬롯
- [ ] Windows 코드 서명(SmartScreen), Linux는 서명된 `.sha256`
- [ ] 카탈로그 만료·롤백 방어(§3.1)

### 6.2 원격 취득
- [ ] 허용 호스트: `github.com`, `objects.githubusercontent.com` (+ `terra-releases`가 쓰는 CDN 호스트). 그 밖은 거부
- [ ] HTTPS 전용, 리다이렉트 상한, 크기 상한, 타임아웃, 이어받기 시 해시 재검증
- [ ] 압축 해제: zip-slip, 심볼릭 링크, 절대경로, 장치 이름(Windows) 거부
- [ ] 스테이징 ACL, 검증 전 실행 금지, 원자적 배치
- [ ] 버전 단조 증가(다운그레이드는 명시 플래그)
- [ ] 프록시·사설 미러 설정은 카탈로그가 아니라 **로컬 설정**으로만(카탈로그가 호스트를 바꿀 수 없다)

### 6.3 로컬 웹 UI
- [ ] §5의 토큰·Host/Origin·CSRF·CSP·세션 단일화
- [ ] 승격 프로세스 명령 열거형과 입력 검증
- [ ] 관리자 암호·등록 코드가 URL·로그·이벤트 스트림에 남지 않는다
- [ ] 같은 기계 다른 사용자 프로세스가 포트를 가로채는 경우(토큰 + 소켓 소유자 확인)

### 6.4 엔진
- [ ] **R-1**: 제거 중간 실패에도 유지관리 바이너리를 지킨다(복구 불가 상태는 손으로 고치게 만드는 길이다)
- [ ] **R-4**: Tree 등록 해제 제안 조건을 제품 이름이 아니라 역할 기준으로
- [ ] **U-1**: `update`에서 번들에서 빠진 모듈의 설치 기록 정리
- [ ] **IN-TODO-01**: 켠 옵션(`fleet_host`, `ssh_server`)을 끄는 경로
- [ ] 기존 무결성 검사(설치 후 손댄 파일 거부)·다운그레이드 거부 유지

## 7. 배포 파이프라인

1. `build-release`가 코어 아카이브·`.sha256`·v3 카탈로그를 만든다(modules 태그 해석 포함)
2. `release.yml`이 카탈로그에 서명한다
3. 기존 순서 유지: draft → 검증 → 노트 → 공개
4. 공개 단계에서 `terra-releases`의 같은 태그 릴리스로 자산을 게시한다(게시용 토큰은 사용자가 비밀값으로 등록)
5. 설치기·카탈로그의 안정 주소(`.../releases/latest/download/release-index.json`)는 README에 둔다

Terra 저장소 릴리스는 내부 기록·검증용으로 계속 둔다. 사용자 다운로드는 `terra-releases`만 가리킨다.

## 8. 마이그레이션

- 기존 설치(0.5.x)는 `install-manifest.json`이 있으므로 새 설치기가 유지관리 모드로 인식해야 한다. 원장 스키마는 바꾸지 않는다
- `terra-setup update`는 카탈로그에서 새 코어·모듈을 받아 같은 검증 경로로 교체한다
- 레거시 단일 제품(`terra-leaf`/`terra-tree`) 제거 경로는 유지한다

## 9. 열린 결정

| ID | 질문 | 권고 |
| --- | --- | --- |
| Q-1 | `lab.stellaxia.node-gui` `.tmod`가 modules 릴리스에 실제로 올라가 있는가 | **확인함(2026-10-06).** `v2026.10.05`에 `lab.stellaxia.node-gui-0.2.0.tmod`가 있다(플랫폼 접미사 없는 단일 자산). 그러나 modules `main`의 모듈은 **0.4.0**(maingui `a884226`)이라 릴리스가 **두 단계 낡았다**. 설치기가 최신 GUI를 받으려면 modules가 새 태그를 찍어야 하고, Terra `bundled-modules.json`의 핀도 올려야 한다. 또 이 모듈이 `bundled-modules.json`에 선언돼 있는지는 확인하지 못했다 |
| Q-8 | modules 릴리스가 이미 내는 `modules.json`(schemaVersion 1: 모듈별·타깃별 자산 이름과 sha256)을 카탈로그 `modules[]`의 원천으로 쓰나 | 쓴다. 빌드가 이 파일을 읽어 URL을 붙이고 `required`·`roles`를 `bundled-modules.json`에서 얻어 v3에 새긴다. 해시를 새로 계산하지 않는다 |
| Q-9 | modules의 `.tmod` 서명(`pack --sign-key`)을 언제 켜나 | modules `release.yml`은 "검증하는 쪽에 신뢰 앵커가 없는 동안의 서명은 장식"이라며 미뤄 뒀다(Q-13). 이 설계의 루트 키가 그 앵커가 될 수 있다. 1단계는 카탈로그 sha256으로 덮고, 2단계에서 모듈 서명을 같은 루트 아래에 둔다 |
| Q-2 | 서명 방식 | Ed25519 분리 서명(minisign 호환). 의존이 가볍고 설치기에 넣기 쉽다 |
| Q-3 | 카탈로그 서명을 CI가 하는가, 사람이 하는가 | 서명 전용 키를 CI 비밀값으로 두되 키 회전 절차를 먼저 쓴다 |
| Q-4 | modules `.tmod`의 자체 서명·digest를 설치기가 어디까지 검증하나 | 카탈로그 sha256이 1차, `.tmod` 내부 무결성이 2차 |
| Q-5 | Windows 코드 서명 인증서 보유 여부 | 없으면 SmartScreen 경고를 문서화하고 취득 계획을 세운다 |
| Q-6 | `terra-releases` 릴리스 자산 보존 정책(오래된 태그) | 최근 N개 + 서명된 목록은 영구 |
| Q-7 | 설치기가 노드 등록 코드(`tnc_…`)를 받아 등록할 수 있게 할까 | 지금은 `account`·`token`·`auto`·`none`만. 코드는 설치 뒤 메인 GUI에서 쓴다(maingui Q-1: 비밀번호를 브라우저에 넣지 않는 것이 코드 방식의 요점). 설치기에서 받으면 비밀번호 입력 없이 붙을 수 있어 이득이므로 엔진 쪽 가능 여부부터 확인 |

## 10. 구현 순서

1. Q-1 확인, 카탈로그 v3 스키마 확정
2. 엔진 선행 정리(§6.4) — 새 UI가 결함 있는 엔진 위에서 처음 나가지 않게
3. 취득·검증 모듈(`internal/fetch`) — 서명·해시·압축 해제, 단위·적대 입력 시험
4. 로컬 웹 UI 서버 + 이벤트 스트림, 비승격/승격 분리
5. 화면 구현(브리프 기준), 기존 ps1·Linux 마법사 제거
6. CI·게시 파이프라인, 실기계 3플랫폼 실측
