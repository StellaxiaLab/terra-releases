---
title: "설치기 GUI 글꼴 목록"
doc_type: "design-handoff"
scope: "project"
target: "terra-setup"
status: "active"
version: "v0.1"
last_updated: "2026-10-06"
language: "ko-KR"
---

# 설치기 GUI 글꼴 목록

시안은 Google Fonts `<link>`로 불러온다. 구현은 아래 파일을 woff2로 바이너리에 임베드한다(CSP로 외부 차단). 관련: [[installer-gui-design-review]] DR-9.

| 글꼴 | 굵기 | 쓰임 |
| --- | --- | --- |
| Noto Sans KR | 400 | 본문 보조 문장 |
| Noto Sans KR | 500 | 라벨, 세그먼트 버튼 |
| Noto Sans KR | 600 | 버튼, 카드 제목, 단계 표시(현재) |
| Noto Sans KR | 700 | 화면 제목(h1), 강조 문장, 선택된 세그먼트 |
| JetBrains Mono | 500 | 경로, 명령, 버전, 칩, 모듈 id, 지문 |
| JetBrains Mono | 600 | 요약 타일의 큰 숫자, `v2026.10.05` 같은 강조 값 |

- 대체 순서: `'Noto Sans KR', system-ui, sans-serif` / `'JetBrains Mono', ui-monospace, Consolas, monospace`.
- 한글 글리프 범위가 커서 Noto Sans KR은 사용 글리프만 서브셋하는 방안을 구현에서 검토한다(굵기 4개 × 서브셋).
- 워드마크 `TERRA`는 Noto Sans KR 600, 자간 `.42em`.
