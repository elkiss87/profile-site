# profile-site 기획

작성 시작: 2026-07-29

개인 프로필 / 링크 페이지. 인스타그램, 블로그 등의 링크를 한 페이지에 모으는
정적 사이트. **모바일 화면을 우선**으로 만든다.

---

## 문서 안내

이 문서는 **무엇을 할지**만 담는다. 세부 내용은 나눠져 있다.

| 문서 | 내용 |
|---|---|
| [DEPLOY.md](DEPLOY.md) | 배포 절차 — Pages 켜기부터 HTTPS까지. **실제로 할 때 보는 문서** |
| [DOMAIN.md](DOMAIN.md) | 도메인·DNS 개념, apex 문제, 레지스트라 선택 근거 |
| [STACK.md](STACK.md) | 기술 스택 검토와 결정 근거 |
| [TROUBLESHOOTING.md](TROUBLESHOOTING.md) | 실제로 막혔던 지점과 진단 방법 |

---

## 1. 1차 목표 — 달성

> **`https://elkiss.me` 를 열면 "Hello World" 가 뜬다.**

예쁠 필요도, 내용이 있을 필요도 없었다. 목적은 **파이프라인 검증**이었다.
도메인, DNS, GitHub Pages, HTTPS 네 개를 한 번 끝까지 연결해보는 것.

**완료 조건**

- [x] `elkiss.me` 도메인 소유
- [x] DNS 레코드가 GitHub Pages를 가리킴
- [x] `https://elkiss.me` 접속 시 페이지가 뜸
- [x] 자물쇠 아이콘 정상 (HTTPS, 인증서 경고 없음)
- [x] `http://` 로 들어가도 `https://` 로 자동 전환됨

**이제부터는 HTML 파일만 고치면 되고, 인프라는 다시 건드리지 않는다.**

---

## 2. 현재 상태

| 항목 | 상태 |
|---|---|
| 도메인 | `elkiss.me` (Cloudflare Registrar, 연 $16 선) |
| DNS | Cloudflare. `@`, `www` 둘 다 CNAME → `elkiss87.github.io`, DNS only |
| 호스팅 | GitHub Pages (`main` / root) |
| HTTPS | Let's Encrypt, Enforce HTTPS 적용 |
| 내용 | `index.html` — Hello World 한 줄. CSS 없음 |

**결정된 것**

- public 저장소, MIT 라이선스
- 순수 HTML 단일 파일 (빌드 도구·프레임워크 없음) → 근거는 [STACK.md](STACK.md)
- 모바일 우선
- 도메인 하나를 서브도메인으로 나눠 쓴다 (프로필=루트, 블로그=`blog.`)
- 프레임워크 결정은 블로그 저장소로 미룬다

---

## 3. 다음 할 일

인프라가 끝났으니 이제 내용이다.

1. [ ] 페이지에 넣을 내용 정하기 — 표시할 이름, 한 줄 소개, 링크 목록
2. [ ] 디자인 정하기 — 색, 폰트, 레이아웃 (모바일 우선)
3. [ ] 페이지 제작

---

## 4. 나중에

- 블로그 — 별도 저장소 + `blog.elkiss.me`. 절차는 [DEPLOY.md](DEPLOY.md) 참고
- 방문자 분석 / 카운터 / 댓글 — 필요해지면 그때

지금 정할 필요가 없는 것을 미리 정하지 않는다.
