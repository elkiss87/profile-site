# 배포 삽질 기록

작성: 2026-07-30

`elkiss.me` 를 GitHub Pages에 연결하면서 막혔던 지점들. 원인과 진단 방법을 남긴다.
나중에 `blog.elkiss.me` 를 붙일 때 같은 데서 또 막히지 않으려고 적는다.

절차는 [DEPLOY.md](DEPLOY.md), 개념은 [DOMAIN.md](DOMAIN.md) 에 있다.
**여기서 얻은 교훈은 DEPLOY.md 의 순서에 반영해두었다.**

---

## 요약

| # | 증상 | 진짜 원인 | 오해했던 것 |
|---|---|---|---|
| 1 | `NotServedByPagesError` | DNS 레코드를 아직 안 넣음 | "전파 대기 중인가?" |
| 2 | `elkiss87.github.io` 도 안 열림 | `CNAME` 파일 때문에 정상 리다이렉트 중 | "Pages가 고장났나?" |
| 3 | `DNS Check in Progress` 에서 멈춤 | `www` 만 넣고 `@` 를 빠뜨림 | "Cloudflare가 느린가?" |

**셋 다 기다려서 해결되는 문제가 아니었다.** 매번 "전파 문제겠지" 하고 기다렸는데
실제로는 설정이 빠져 있었다. DNS 문제는 일단 조회부터 해보고 판단해야 한다.

---

## 1. NotServedByPagesError

```
Both elkiss.me and its alternate name are improperly configured
Domain does not resolve to the GitHub Pages server. (NotServedByPagesError)
```

### 원인

도메인은 샀지만 **DNS 레코드를 하나도 안 넣은 상태**였다.

### 진단

로컬 캐시를 피하려고 공용 리졸버로 직접 조회한다.

```bash
nslookup -type=NS elkiss.me 1.1.1.1
nslookup -type=A elkiss.me 1.1.1.1
```

결과가 이랬다.

- **NS** → `dion.ns.cloudflare.com`, `isabel.ns.cloudflare.com` → 도메인 등록과 위임은 정상
- **A** → 레코드 없음 (SOA만 반환)

즉 "Cloudflare가 담당인 건 맞는데, 알려줄 내용이 없는" 상태였다.

### NXDOMAIN 과 NODATA 구분

진단할 때 유용한 차이다.

| 응답 | 의미 |
|---|---|
| **SOA 반환** (NODATA) | 이름은 존재하는데 그 타입의 레코드가 없다 |
| **`does not exist`** (NXDOMAIN) | 이름 자체가 없다 |

`elkiss.me` 는 SOA(구역은 있음), `www.elkiss.me` 는 NXDOMAIN(이름 없음)이었다.
이 둘을 구분하면 "구역 설정이 잘못됐나"와 "레코드만 빠졌나"를 바로 가를 수 있다.

---

## 2. `elkiss87.github.io` 주소도 안 열림

커스텀 도메인이 안 되니 원래 주소로 확인하려 했는데 그것도 안 열렸다.
Pages 자체가 고장났다고 오해했다.

### 원인

**정상 동작이었다.** 커스텀 도메인을 설정하면 GitHub이 저장소 루트에 `CNAME` 파일을
자동 생성하고, 그때부터 `*.github.io` 로 오는 요청을 커스텀 도메인으로 301 리다이렉트한다.
그런데 그 도메인이 아직 DNS에 없으니 브라우저가 도착을 못 했다.

### 진단

```bash
curl.exe -sS -I "https://elkiss87.github.io/profile-site/"
```

```
HTTP/1.1 301 Moved Permanently
Server: GitHub.com
Location: http://elkiss.me/
```

리다이렉트를 따라가면 `Could not resolve host: elkiss.me` 에서 끊긴다.

### 얻은 것

`Server: GitHub.com` 이 찍힌 301이 왔다는 건 **Pages가 이미 정상 동작 중**이라는 뜻이다.
꺼져 있으면 404가 온다. 이걸로 남은 변수를 DNS 하나로 좁힐 수 있었다.

> `Location` 이 `http://` 인 것도 정상이다. Enforce HTTPS를 켜기 전이라 그렇다.

### 교훈

**커스텀 도메인을 걸고 나면 `*.github.io` 주소로는 확인할 수 없다.**
확인 순서를 반대로 잡아야 한다 — Pages를 먼저 켜서 `*.github.io` 로 동작을 확인하고,
그다음에 커스텀 도메인을 얹는다.

---

## 3. DNS Check in Progress 에서 멈춤

레코드를 넣었는데도 GitHub 검사가 안 끝났다. Cloudflare 반영이 느린 줄 알았다.

### 원인

**두 줄 중 `www` 한 줄만 넣고 `@`(루트)를 빠뜨렸다.**

### 진단

```bash
nslookup -type=A www.elkiss.me 1.1.1.1   # 정상: IP 4개까지 따라감
nslookup -type=A elkiss.me 1.1.1.1        # SOA만 반환 = 레코드 없음
```

### 왜 www 만으로는 안 되나

`CNAME` 파일에 적힌 이름이 `elkiss.me`(루트)다. **GitHub이 검사하는 건 그 이름 하나**다.
`www` 가 아무리 잘 잡혀도 루트가 비어 있으면 통과하지 못한다.

### 필요한 레코드

| Type | Name | Target | Proxy |
|---|---|---|---|
| CNAME | `@` | `elkiss87.github.io` | **DNS only** (회색 구름) |
| CNAME | `www` | `elkiss87.github.io` | **DNS only** (회색 구름) |

`@` 를 입력창이 받지 않으면 `elkiss.me` 를 그대로 입력해도 된다.
저장 후 목록에 `elkiss.me` 로 표시되는 것이 정상이다.

### 확인 방법

```bash
nslookup elkiss.me 1.1.1.1
```

루트 조회인데 **A 레코드(IP)가 나오면 성공**이다. CNAME을 걸었는데 IP가 나오는 것은
Cloudflare의 **CNAME flattening** 이 동작한 결과다. `www` 는 CNAME 그대로 나오고
루트만 IP로 나오는 차이를 확인할 수 있다.

---

## 진단에 쓴 것들

| 목적 | 명령 |
|---|---|
| 로컬 캐시 우회 조회 | `nslookup <name> 1.1.1.1` |
| 레코드 타입 지정 | `nslookup -type=NS elkiss.me 1.1.1.1` |
| 리다이렉트 확인 | `curl.exe -sS -I <url>` |

**공용 리졸버(`1.1.1.1`, `8.8.8.8`)를 명시해서 조회하는 게 핵심이다.**
그냥 조회하면 로컬/ISP 캐시가 껴서 오래된 답이 나올 수 있다.

브라우저로 확인할 때는 시크릿 창을 쓰거나, PC와 폰(LTE)에서 각각 확인한다.

---

## git 쪽에서 있었던 일

**커스텀 도메인을 설정하면 원격에 커밋이 하나 생긴다.** GitHub이 `CNAME` 파일을
자동 커밋하기 때문이다. 로컬에서 작업 중이었다면 히스토리가 갈라진다.

```
원격:  1153009 → aad5bf1 (Create CNAME)
로컬:  1153009 → 0cb958c (작업 커밋)
```

이 상태로 push하면 거부당한다. rebase로 정리한다.

```bash
git pull --rebase origin main
```

**민감 정보 점검에서 걸린 것**: 파일 내용은 깨끗했는데, **커밋 작성자 이메일**이
개인 주소로 되어 있었다. 파일이 아니라 커밋 메타데이터라 파일만 훑으면 놓친다.
public 저장소에서는 그대로 공개된다. GitHub의 noreply 주소로 바꾸면 된다.

이 내용은 [CLAUDE.md](../CLAUDE.md) 의 커밋 전 점검 항목에 반영했다.

---

## 이 기록에서 나온 결론

**단계마다 확인하고 넘어간다. 안 되면 기다리기 전에 조회부터 한다.**

이 두 줄이 세 번의 삽질에서 나온 전부다. [DEPLOY.md](DEPLOY.md) 의 맨 위에
원칙으로 적어두었고, 절차의 순서도 그에 맞게 바꿨다.

blog 서브도메인을 붙일 때는 [DEPLOY.md](DEPLOY.md) 의 순서를 그대로 따르면 된다.
서브도메인은 apex 문제가 없어서 더 단순하다.
