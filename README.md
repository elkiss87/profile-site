# profile-site

개인 프로필 & 링크 페이지 (link in bio).

인스타그램, 블로그 등 흩어져 있는 링크를 한 페이지에 모아두는 사이트입니다.
모바일에서 보기 좋은 것을 우선으로 합니다.

**https://elkiss.me**

## 상태

인프라와 1차 내용이 끝났습니다. 자리표시자는 남아 있지 않습니다.

- [x] GitHub Pages 배포
- [x] 커스텀 도메인 연결 (`elkiss.me`, HTTPS)
- [x] 페이지 뼈대 제작 (모바일 우선)
- [x] 프로필 내용 (역할, 현재 상태, 이력)
- [x] 링크 연결 (Instagram, GitHub, Blog)
- [ ] 3D 오브젝트 적용 ([RESEARCH.md](docs/RESEARCH.md))

## 구조

빌드 도구와 프레임워크 없이 **순수 HTML 파일 하나**로 만듭니다.
CSS는 같은 파일 안에 두고, 아이콘은 인라인 SVG, 폰트는 시스템 폰트를 씁니다.
외부 요청은 없습니다. 이 파일 하나만 받으면 페이지가 완성됩니다.

```
index.html    페이지 전체 (HTML + CSS)
CNAME         커스텀 도메인 설정
docs/         기획과 기록
```

결정 근거는 [docs/STACK.md](docs/STACK.md) 에 있습니다.

## 문서

| 문서 | 내용 |
|---|---|
| [PLAN.md](docs/PLAN.md) | 기획, 진행 상황, 페이지 규칙 |
| [DEPLOY.md](docs/DEPLOY.md) | 배포 절차 |
| [DOMAIN.md](docs/DOMAIN.md) | 도메인·DNS 개념 |
| [STACK.md](docs/STACK.md) | 기술 스택 결정 근거 |
| [RESEARCH.md](docs/RESEARCH.md) | UI 자료 조사와 채택 여부 |
| [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) | 막혔던 지점과 진단 방법 |
| [LOG.md](docs/LOG.md) | 작업 일지 |

## 로컬에서 보기

빌드가 없으므로 `index.html` 을 브라우저로 열면 됩니다.

```bash
start index.html
```

모바일 화면은 `F12` → `Ctrl+Shift+M` (기기 툴바)으로 확인합니다.

## 라이선스

[MIT](LICENSE)
