---
title: "설치기 GUI 시안 인계 — 상태 변형 · 컴포넌트 · 문구표"
doc_type: "design-handoff"
scope: "project"
target: "terra-setup"
status: "active"
version: "v0.1"
last_updated: "2026-10-06"
language: "ko-KR"
---

# 설치기 GUI 시안 인계

[[installer-gui-design-review]] §4 요청에 대한 답이다. 파일 구조는 `design/installer-gui/*.dc.html` + `canvas.json` 그대로이고, 구현은 화면 ID로 참조한다. 시안 안의 숫자·경로·버전은 예시다.

## 1. 검토 항목 반영

| ID | 반영 |
| --- | --- |
| DR-1 | S4·S5·S6·S8·V3·M0·M3에 Linux/Windows 전환(우상단). Windows: `%LOCALAPPDATA%\Terra\Leaf\`(Tree는 `\Tree\`), `winget` 명령, UAC 안내, 사용자 PATH, 서비스·예약 작업. **macOS는 이번 범위에서 제외**(브리프 §1이 Windows·Linux) |
| DR-2 | 출처 칩 `modules 릴리스 · lab.stellaxia.*` / `동봉 · io.terra.*`. "Tree 레지스트리"는 S3 상단 "받는 곳"에만 |
| DR-3 | `:focus-visible` 2px 외곽선(`#7aa7ff`, 오프셋 2px). 테두리 제거 규칙은 `border`만 건드리고 외곽선·`box-shadow`는 건드리지 않는다. 선택 카드는 안쪽 2px 윤곽. 세그먼트·스위치·체크박스는 `<button>`(`aria-pressed` / `role="switch"` / `role="checkbox"`) |
| DR-4 | M3 확인 문구 `terra`. 붙여넣기 허용, 대소문자 무시(구현 확인 필요) |
| DR-5 | S6 시스템 영역: 실제로 건드리는 것만 색, 나머지는 흐림. Linux는 PATH·서비스, Windows는 사용자 PATH·서비스·예약 작업 |
| DR-6 | V3 행동은 하단 버튼 한 곳. 카드는 설명만 |
| DR-7 | M0 `v2026.10.05` |
| DR-8 | S7 하단은 취소만 |
| DR-9 | [[fonts]] |
| DR-10 | V1 전체 지문 + 복사 버튼, "자세히"에 키 유효 기간 · 서명 시각 · 받은 서명 키. 지문 표기 길이는 구현 확인 필요 |
| DR-11 | V5 만료된 카탈로그, V6 옛 버전 목록 거부 추가 |
| DR-12 | E1 "이어서 받기"(검증된 파일은 다시 받지 않음) / "처음부터 다시". 이어받기 가능 여부는 구현 확인 필요 |
| DR-13 | N1 좁은 화면(≤ 800px): 좌측 단계줄이 상단 한 줄로 접히고 현재 단계와 `n / 8`만 표시 |
| DR-14 | 모든 화면에 `prefers-reduced-motion` 규칙 |
| DR-15 | 제외 확인(콘솔 문구는 구현이 쓴다) |

## 2. 화면별 상태 변형

| 화면 | 변형 | 시안 위치 |
| --- | --- | --- |
| S1 역할 | Tree만 / Leaf만 / 둘 다 / 없음(다음 비활성) | `Main`, 카드 토글 |
| S2 연결 | 자동(찾지 못함) / 계정 / 토큰 / 나중에 등록 | `S2Connect`, 세그먼트 |
| S3 모듈 | 기본 / 오프라인 | `S3Modules`, `V2Offline` |
| S4 의존성 | 승인 / 건너뛰기 / 이미 있음 | `S4Deps` |
| S5 옵션 | docker 없음 경고 / ssh-server 비활성 | `S5Options` |
| S6 요약 | OS별 | `S6Summary` |
| S7 진행 | 등록 하위 단계 진행 중 | `S7Progress` |
| S8 완료 | Tree 설치 / Leaf 등록됨 / Leaf 등록 안 함 / 등록 지연 | `S8Done` |
| E1 | 받기 실패(이어서·처음부터) | `E1Error` |
| M0~M4 | 홈 / 변경점 / 점검 / 제거(보존·완전삭제) / 등록 해제 | `M*` |
| V1~V6 | 서명 실패 / 오프라인 / 승격 거부 / 이미 실행 중 / 만료 / 되돌림 거부 | `V*` |

**빈 상태·로딩:** 시안에 별도 화면이 없는 곳은 S3 목록 로딩(스켈레톤 행), S2 "연결 확인" 대기, M2 점검 중이다. 구현이 같은 컴포넌트의 로딩 변형으로 처리하고, 필요하면 디자인 세션이 추가한다.

## 3. 컴포넌트

| 컴포넌트 | 변형 |
| --- | --- |
| 창 셸 | 헤더(로고·워드마크·서명 칩·카탈로그 버전) / 좌측 단계줄 / 본문 / 푸터 |
| 단계줄 | 완료(초록 체크) / 현재 / 대기 / 좁은 화면 접힘 |
| 체크 카드 | 선택 / 해제 (S1) |
| 세그먼트 | 선택 / 해제 / 흐림 |
| 스위치 | 켬 / 끔 / 비활성 |
| 입력 | 기본 / 포커스 / 가림(암호·토큰) |
| 모듈 행 | 필수 / 선택 / 받지 않음 / 확인 못 함 / 없음 |
| 칩 | 기본 / ok / wn / bad / info / 역할(Tree·Leaf) / 필수 / 흐림 |
| 알림 박스 | 경고(노랑) / 오류(빨강) / 안내(파랑) |
| 명령 상자 | 한 줄 / 여러 줄 (복사 가능) |
| 단계 목록 | 완료 / 진행 중 / 대기 / 실패 / 하위 단계 |
| 진행 막대 | 진행 중 / 줄임 모션 |
| 오류 카드 | 원인 + 상태(`rolled_back`) + 다음 행동 |
| 확인 대화상자 | 취소 롤백 확인, 제거 확인 문구 입력 |
| 버튼 | 기본 / primary / 위험 / 비활성 / 포커스 |

## 4. 문구표 (정본은 이 표, 시안 문구는 예시)

| 키 | 한국어 | 화면 |
| --- | --- | --- |
| role.tree.sub | 새 Tree 만들기 | S1 |
| role.leaf.sub | 기존 Tree에 붙기 | S1 |
| role.both.note | Tree를 먼저 설치하고 Leaf를 이어서 설치합니다. 같은 컴퓨터의 Tree라서 Leaf 연결은 자동으로 이어져요. | S1 |
| role.none.note | 역할을 하나 이상 골라야 계속할 수 있어요. | S1 |
| enroll.auto.notfound | 이 컴퓨터에서 Tree를 찾지 못했어요. Tree도 함께 설치하면 자동으로 연결됩니다. | S2 |
| enroll.self.warn | 이 주소는 이 컴퓨터 자신입니다. 다른 컴퓨터의 Tree에 붙으려면 그 Tree의 주소로 바꾸세요. | S2 |
| enroll.none.info | 등록 전에는 이 컴퓨터의 브라우저에서 할 수 있는 일이 없습니다. 등록은 설치 뒤 열리는 화면에서 등록 코드로 해요. | S2 |
| enroll.secret.hint | 암호는 이 화면 밖에 저장하지 않고, 주소·로그·진행 기록에도 남기지 않아요. | S2 |
| module.source.release | modules 릴리스 · lab.stellaxia.* | S3 |
| module.source.bundled | 동봉 · io.terra.* | S3 |
| module.node-gui.required | 이 모듈이 없으면 설치 뒤 첫 화면이 비어 있어요. | S3 |
| module.offline | 코어는 설치하고, 확인하지 못한 모듈은 나중에 채웁니다. | V2 |
| module.status.unknown / .absent | 확인 못 함 / 없음 | V2 |
| dep.skip.docker | 건너뛰면 fleet-host를 쓸 수 없어요. Master가 이 노드에 놓은 컨테이너 좌석을 돌리지 못합니다. | S4 |
| opt.fleet.nodocker | docker를 찾지 못했어요. 이대로 켜면 데몬이 계속 다시 시작됩니다. 이전 단계에서 docker 설치를 승인하거나 이 옵션을 끄세요. | S5 |
| summary.areas.hint | 색이 있는 것만 바뀌고, 흐린 것은 이번 설치에서 건드리지 않아요. 옵션을 바꾸면 달라져요. | S6 |
| progress.cancel.hint | 취소하면 지금까지 바뀐 것을 원래대로 돌려요. | S7 |
| progress.slow.hint | 3분을 넘기면 원인을 알려 드려요. 서비스가 아니라 직접 띄운 Daemon이면 재시작이 끝나지 않을 수 있어요. | S7 |
| done.leaf.pending | 3분 안에 Tree가 응답하지 않았어요. 설치는 그대로 남아 있습니다. | S8 |
| error.download.resume | 이미 받아서 검증한 N개는 다시 받지 않아요. 이어서 받기는 나머지만 받고, 처음부터 다시는 받은 파일을 지우고 모두 새로 받아요. | E1 |
| update.removed.warn | 제거됩니다 — 설치 기록까지 지워집니다. | M1 |
| check.overwrite.warn | 직접 수정한 파일은 덮어쓰기 전에 한 번 더 확인합니다. | M2 |
| remove.confirm | 계속하려면 terra 를 입력하세요 | M3 |
| remove.confirm.hint | 붙여넣기도 돼요 · 대소문자는 구분하지 않아요 | M3 |
| sig.fail.title | 카탈로그를 받지 않았습니다 | V1 |
| catalog.stale.title | 이 목록은 오래됐습니다 | V5 |
| catalog.rollback.title | 옛 버전 목록이라 받지 않았습니다 | V6 |
| elevation.denied.title | 관리자 권한이 승인되지 않았어요 | V3 |
| session.running.title | 다른 설치기가 이미 실행 중이에요 | V4 |

## 5. 새 토큰

새 토큰은 없다. 브리프 §6 토큰만 쓴다. 추가한 값은 포커스 링 색 `#7aa7ff`(기존 info 색)뿐이다.

## 6. 데이터 계약 추가 요청 (브리프 §8 기준)

| 요청 | 화면 |
| --- | --- |
| 카탈로그 `expires_at` 외에 `issued_at`, 직전 설치 버전 | V5, V6 |
| 서명 키 지문 전체, 키 유효 기간, 서명 시각, 받은 파일의 서명 키 지문 | V1 |
| 이어받기 가능 여부와 받은 파일 수/전체 수 | E1 |
| 단계별 "건드리는 영역" 목록(PATH, 서비스, 방화벽, 예약 작업) | S6 |
| OS 종류와 승격 방식(`sudo` / `uac`) | S4, S6, V3 |
