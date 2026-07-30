# profile-site

개인 프로필 & 링크 페이지 (link in bio).

인스타그램, 블로그 등 흩어져 있는 링크를 한 페이지에 모아두는 사이트입니다.
모바일에서 보기 좋은 것을 우선으로 합니다.

**https://elkiss.me**

## 상태

인프라는 끝났고 페이지 내용을 채우는 중입니다.
지금은 `Under construction` 표시만 나옵니다.

- [x] GitHub Pages 배포
- [x] 커스텀 도메인 연결 (`elkiss.me`, HTTPS)
- [x] 페이지 뼈대 제작 (모바일 우선)
- [ ] 내용 정하기 (이름, 소개, 링크 목록)
- [ ] 내용 채우고 공개

## 구조

빌드 도구와 프레임워크 없이 **순수 HTML 파일 하나**로 만듭니다.
CSS는 같은 파일 안에 두고, 아이콘은 인라인 SVG, 폰트는 시스템 폰트를 씁니다.
외부 요청은 GitHub 아바타 하나뿐입니다.

```
index.html    페이지 전체 (HTML + CSS)
CNAME         커스텀 도메인 설정
docs/         기획과 기록
```

결정 근거는 [docs/STACK.md](docs/STACK.md) 에 있습니다.

## 문서

| 문서 | 내용 |
|---|---|
| [PLAN.md](docs/PLAN.md) | 기획과 진행 상황 |
| [DEPLOY.md](docs/DEPLOY.md) | 배포 절차 |
| [DOMAIN.md](docs/DOMAIN.md) | 도메인·DNS 개념 |
| [STACK.md](docs/STACK.md) | 기술 스택 결정 근거 |
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
