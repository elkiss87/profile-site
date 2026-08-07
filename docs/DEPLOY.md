# 배포 절차

GitHub Pages에 올리고 커스텀 도메인을 연결하는 순서.
개념 설명은 [DOMAIN.md](DOMAIN.md), 실제로 막혔던 지점은
[TROUBLESHOOTING.md](TROUBLESHOOTING.md) 에 있다.

---

## 원칙

**단계마다 확인하고 넘어간다.**

한 번에 여러 개를 설정하고 마지막에 확인하면, 안 될 때 어디가 원인인지 알 수 없다.
DNS인지 Pages인지 인증서인지 구분이 안 되면 손댈 곳을 못 찾는다.
각 단계 끝에 확인 방법을 적어두었다. **통과하지 못하면 다음으로 넘어가지 않는다.**

**안 되면 기다리기 전에 조회부터 한다.**

"DNS 전파 중이겠지" 하고 기다리는 것이 가장 흔한 함정이다.
전파 문제인지 설정 누락인지는 조회 한 번이면 갈린다. 기다리는 건 그다음이다.

---

## 순서

```
1. Pages 켜기        → *.github.io 로 확인
2. DNS 레코드        → 조회로 확인
3. 커스텀 도메인 등록 → GitHub DNS 검사 통과 확인
4. Enforce HTTPS     → 최종 확인
```

**커스텀 도메인 등록(3번)을 마지막 쪽에 두는 것이 중요하다.**
등록하는 순간 `*.github.io` 주소가 커스텀 도메인으로 리다이렉트되어
**확인 창구가 사라지기 때문이다.** DNS를 먼저 붙여두면 이 구간이 생기지 않는다.

---

## 1. Pages 켜기

**전제: 내용이 저장소에 push되어 있어야 한다.**
`index.html` 이 원격에 없으면 Pages를 켜도 보여줄 것이 없다.

```bash
git push
```

저장소 `Settings` → `Pages`

| 항목 | 값 |
|---|---|
| Source | `Deploy from a branch` |
| Branch | `main` / `(root)` |

**커스텀 도메인은 아직 입력하지 않는다.**

> **처음 켤 때는 이 방식으로 시작한다.** 설정 두 줄이면 끝이라 파이프라인 검증이
> 빠르다. 워크플로를 얹은 채로 사이트가 안 뜨면 원인이 DNS인지 빌드인지 구분되지 않는다.
>
> 이 저장소는 나중에 **GitHub Actions 방식으로 전환했다.** 배경과 절차는
> [5. Actions 방식으로 전환](#5-actions-방식으로-전환) 참고.

### 확인

```bash
curl.exe -sS -I "https://<사용자>.github.io/<저장소>/"
```

`200` 이 나오고 내용이 보이면 통과. **여기서 배포 파이프라인이 검증된 것이다.**
이후 문제가 생기면 Pages가 아니라 도메인 쪽을 의심하면 된다.

---

## 2. DNS 레코드

Cloudflare 대시보드 → 해당 도메인 → `DNS` → `Records` → `Add record`

| Type | Name | Target | Proxy | 필수 |
|---|---|---|---|---|
| CNAME | `@` | `<사용자>.github.io` | **DNS only** | **필수** |
| CNAME | `www` | `<사용자>.github.io` | **DNS only** | 권장 |

### `@` 가 필수인 이유

**GitHub이 검사하는 것은 `CNAME` 파일에 적힌 이름 하나다.**
루트 도메인(`elkiss.me`)을 쓴다면 검사 대상은 `@` 다.
`www` 만 넣으면 아무리 잘 잡혀도 검사를 통과하지 못한다.

`www` 는 "www 붙여서 들어오는 사람"을 위한 것이다. 없어도 사이트는 뜬다.

### 주의

- **Proxy status 를 반드시 `DNS only`(회색 구름)로 둔다.**
  Cloudflare 기본값은 Proxied(주황)다. 켜져 있으면 GitHub이 Let's Encrypt
  인증서를 발급받지 못해 HTTPS가 잡히지 않는다.
- Target 끝에 `.` 이나 `https://` 를 붙이지 않는다.
- `@` 를 입력창이 받지 않으면 도메인 전체(`elkiss.me`)를 입력해도 된다.
  저장 후 목록에 도메인 이름으로 표시되는 것이 정상이다.
- 와일드카드(`*`)는 쓰지 않는다. 도메인 탈취 위험이 있다.

### ALIAS 미지원 DNS를 쓰는 경우

`@` 에 CNAME을 걸 수 없다면 A 레코드 4개를 직접 넣는다.
GitHub이 IP를 바꾸면 직접 고쳐야 한다.

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

IPv6까지 넣으려면 AAAA 4개:

```
2606:50c0:8000::153
2606:50c0:8001::153
2606:50c0:8002::153
2606:50c0:8003::153
```

### 확인

로컬 캐시를 피하려고 **공용 리졸버를 명시해서** 조회한다.

```bash
nslookup elkiss.me 1.1.1.1
```

`185.199.1xx.153` 네 개가 나오면 통과.

> CNAME을 걸었는데 조회 결과가 A 레코드(IP)로 나오는 것이 정상이다.
> Cloudflare의 CNAME flattening이 동작한 결과다. 자세한 내용은 [DOMAIN.md](DOMAIN.md).

아무것도 안 나오면 **레코드가 없는 것이다.** 기다려도 생기지 않는다.
Cloudflare 화면으로 돌아가 저장이 됐는지 확인한다.

---

## 3. 커스텀 도메인 등록

`Settings` → `Pages` → `Custom domain` 에 도메인 입력 → `Save`

GitHub이 저장소 루트에 `CNAME` 파일을 자동 생성한다. **이 파일은 지우면 안 된다.**

### git 히스토리가 갈라진다

**자동 생성은 원격에 커밋 하나를 만든다.** 로컬에서 작업 중이었다면 갈라진다.

```
원격:  abc1234 → def5678 (Create CNAME)
로컬:  abc1234 → 작업 커밋
```

이 상태로 push하면 거부당한다. rebase로 정리한다.

```bash
git pull --rebase origin main
```

### 확인

같은 화면에서 `DNS Check in Progress` → 통과를 기다린다.
2번을 제대로 마쳤다면 몇 분 안에 끝난다.

에러가 계속 뜨면 **2번의 조회부터 다시 한다.** 기다리는 것은 답이 아니다.

> 이 시점부터 `*.github.io/<저장소>` 는 커스텀 도메인으로 301 리다이렉트된다.
> 그 주소로는 더 이상 확인할 수 없다. 정상이다.

---

## 4. Enforce HTTPS

DNS 검사를 통과하면 GitHub이 Let's Encrypt 인증서를 발급한다.
발급 중에는 체크박스가 비활성 상태다. **최대 15분 정도 걸린다.**

`TLS certificate is being provisioned` 이 뜨면 진행 중이다. 기다렸다 다시 들어와
**Enforce HTTPS** 를 체크한다.

### 확인

```bash
curl.exe -sS -o NUL -w "%{http_code} %{url_effective}\n" -L "http://elkiss.me/"
```

`http://` 로 요청했는데 최종 URL이 `https://` 이고 `200` 이면 완료.

브라우저에서는 시크릿 창으로 접속해 자물쇠 아이콘에 경고가 없는지 확인한다.
PC와 폰(LTE)에서 각각 보면 캐시 영향을 배제할 수 있다.

---

## 5. Actions 방식으로 전환

**필수가 아니다.** 브랜치 방식으로도 사이트는 정상 동작한다.
아래 중 하나라도 해당되면 그때 전환한다.

- 배포 큐가 꼬여 빌드가 끝나지 않는다
- 올릴 파일을 고르고 싶다 (문서까지 서비스되는 것을 막는다)
- 배포 전에 빌드 단계가 필요해진다

### 절차

**1) 워크플로 파일을 만든다** — `.github/workflows/deploy.yml`.
이 저장소의 것을 그대로 참고하면 된다.

**2) Pages 소스를 바꾼다** — `Settings` → `Pages` → Source 를 `GitHub Actions` 로.

**이 두 번째 단계를 빼먹으면 안 된다.** 워크플로만 추가하고 소스를 그대로 두면
GitHub 자동 워크플로(`pages-build-deployment`)와 커스텀 워크플로가 **둘 다 돌면서
같은 배포 자리를 두고 부딪힌다.** 큐가 꼬인 원인이 대개 이것이다.

### 확인

```bash
gh api repos/<사용자>/<저장소>/pages --jq .build_type
```

`workflow` 가 나오면 전환된 것이다. `legacy` 면 아직 브랜치 방식이다.

```bash
gh run list --limit 5
```

전환 후에는 `pages-build-deployment` 가 새로 뜨지 않아야 한다.

### 알아둘 것

- **올릴 파일을 직접 고른다.** `path: '.'` 로 두면 `README.md`, `docs/` 까지
  도메인에서 그대로 열린다. 필요한 파일만 `_site` 에 모아 올린다
- **`CNAME` 은 함께 올린다.** 커스텀 도메인은 저장소 설정에도 저장되지만,
  아티팩트에 넣어두는 편이 안전하다
- **`cancel-in-progress`** — GitHub 공식 예제는 `false`(진행 중 배포를 지키는 쪽)다.
  `true` 로 두면 큐 꼬임은 막지만 배포 중인 것이 취소된다. 정적 파일 하나짜리라면
  다음 배포가 덮으므로 실질 위험은 없다
- **배포가 느리면 남아 있는 실행부터 본다.** 큐에 걸린 채 상태가 깨진 실행은
  취소·강제취소·삭제가 모두 403/409 로 막힌다. 그런데 그것이 **배포 자리를 붙잡는다.**
  Pages 는 저장소당 배포를 하나씩 처리하기 때문이다. 실제로 그 상태에서 배포가
  6분 39초 걸렸고, 풀린 뒤에는 19초로 끝났다.
  워크플로를 의심하기 전에 `gh run list` 로 끝나지 않은 실행을 확인한다

---

## 서브도메인 추가 (blog 등)

GitHub Pages는 **저장소 하나당 커스텀 도메인 하나**다.
블로그는 별도 저장소 + 서브도메인으로 간다.

| 용도 | 도메인 | 저장소 |
|---|---|---|
| 프로필 | `elkiss.me` | `profile-site` |
| 블로그 | `blog.elkiss.me` | 별도 저장소 |

서브도메인은 apex 문제가 없어서 CNAME 한 줄이면 된다.
**위와 같은 순서를 그대로 따른다.**

1. 새 저장소에 내용 push → Pages 켜기 → `*.github.io` 로 확인
2. Cloudflare에 CNAME 한 줄: `blog` → `<저장소>.github.io`, **DNS only**
3. `nslookup blog.elkiss.me 1.1.1.1` 로 확인
4. Pages에 `blog.elkiss.me` 등록 → 검사 통과 확인
5. Enforce HTTPS

---

## 진단 명령 모음

| 목적 | 명령 |
|---|---|
| DNS 조회 (캐시 우회) | `nslookup <name> 1.1.1.1` |
| 레코드 타입 지정 | `nslookup -type=NS <name> 1.1.1.1` |
| HTTP 응답/리다이렉트 | `curl.exe -sS -I <url>` |
| 리다이렉트 최종 목적지 | `curl.exe -sS -o NUL -w "%{http_code} %{url_effective}\n" -L <url>` |

조회 결과 읽는 법은 [TROUBLESHOOTING.md](TROUBLESHOOTING.md) 의
NXDOMAIN / NODATA 구분을 참고한다.
