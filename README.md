# terra-releases

Terra 의 **배포 전용 저장소**다. 소스 코드는 없다 — 사용자가 내려받는 설치기, 코어 아카이브, 서명된 카탈로그만 이 저장소의 Releases 로 나간다.

> [!NOTE] 상태
> 아직 게시된 릴리스가 없다. 설치기 재설계가 구현되기 전이라 이 저장소는 자리와 규칙만 갖고 있다. 아래 "받는 곳"은 첫 릴리스가 나온 뒤에 유효하다.

## 여기에 올라오는 것

| 자산 | 설명 |
| --- | --- |
| `terra-setup_<버전>_<타깃>` | 부트스트랩 설치기(수 MB). 이것 하나만 받으면 나머지는 설치기가 가져온다 |
| `terra-<제품>_<버전>_<타깃>.{zip,tar.gz}` + `.sha256` | 코어 아카이브(`leaf` · `tree` · `terra`). 에어갭용 풀 번들 변형도 같은 이름 규칙을 따른다 |
| `release-index.json` + `.minisig` | **서명된 카탈로그.** 설치기가 무엇을 어디서 받을지, 해시가 무엇인지를 여기서 읽는다 |
| `release-key.cert.json` + `.minisig` | 릴리스 서명 키의 인증서(루트 키가 서명) |

타깃은 `windows-amd64` · `linux-amd64` · `linux-arm64` 이다.

모듈(`.tmod`)은 이 저장소에 없다. 모듈은 [StellaxiaLab/modules](https://github.com/StellaxiaLab/modules) 릴리스에서 받고, 설치기는 카탈로그에 적힌 해시로 검증한다.

## 신뢰는 어떻게 만들어지나

설치기는 GitHub 나 이 저장소를 믿지 않는다. 믿는 것은 **설치기에 박힌 Terra 루트 공개키**와, 그 키가 인증한 릴리스 키로 서명된 카탈로그다.

```text
루트 키(오프라인) ─ 서명 ─► 릴리스 키 인증서 ─ 서명 ─► release-index.json ─ 해시 ─► 코어·모듈 파일
```

- 서명이나 해시가 하나라도 맞지 않으면 설치기는 **아무것도 받지 않는다.**
- 카탈로그에는 만료일과 최소 설치기 버전이 있어, 오래된 목록을 다시 내미는 공격을 막는다.
- 서명 형식은 Ed25519 이고 [minisign](https://jedisct1.github.io/minisign/) 과 호환되어 직접 검증할 수 있다.

```bash
minisign -Vm release-index.json -p terra-root.pub   # 인증서 단계는 설치기가 자동으로 한다
```

### 루트 공개키 지문

> 발급 전. 첫 릴리스에서 이 자리에 지문을 적는다. 같은 지문이 Terra 문서에도 따로 적혀 있어, 한쪽만 바뀌면 알아챌 수 있다.

## 받는 곳

첫 릴리스 이후 안정 주소는 다음과 같다.

```text
https://github.com/StellaxiaLab/terra-releases/releases/latest/download/release-index.json
```

설치기는 이 주소를 직접 조회한다. 사람이 받을 때는 [Releases](https://github.com/StellaxiaLab/terra-releases/releases) 에서 자기 플랫폼의 `terra-setup` 하나를 받으면 된다.

**Windows 설치기는 코드 서명 인증서가 마련되기 전까지 SmartScreen 경고가 뜰 수 있다.** 그동안은 Releases 에 적힌 `.exe` 의 sha256 과 위 서명 확인법으로 확인한다.

## 문서

- [설치기 재설계 — 원격 취득 부트스트랩과 로컬 웹 UI](docs/installer-design.md) — 구조, 카탈로그 스키마 초안, 보안 작업 목록, 열린 결정
- [설치기 GUI 디자인 브리프](docs/installer-gui-design-brief.md) — GUI 디자인 인계 문서(화면 흐름, 명세, 시각 규칙)

두 문서는 Terra 저장소의 `docs/architecture/operations/` 가 정본이다. 이 저장소의 사본은 공개용으로 내부 문서 링크만 걷어 낸 것이며, 어긋나면 정본이 맞다.

## 이 저장소에 대해

- 릴리스 자산은 Terra 의 릴리스 파이프라인이 올린다. 사람이 손으로 올린 자산은 서명이 없으므로 설치기가 받아들이지 않는다.
- 이 저장소의 `main` 에는 README 와 문서만 둔다. 바이너리를 커밋하지 않는다.
