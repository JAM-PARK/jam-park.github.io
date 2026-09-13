# HANDOFF — 조직 웹사이트 구현 (Google Play 조직 계정 인증용)

> **Standalone 작업 문서.** 이 파일 하나만 들고 새 레포에서 작업을 시작할 수 있게 썼다.
> 원본 레포(`infinite-loop-factory/app-factory`)를 열어 볼 필요는 없다. 필요한 맥락은 모두 여기 있다.
> 새 레포에 `HANDOFF.md` 로 복사해 두고, 끝나면 §9 결과 보고란을 채운다.
>
> - 작성일: 2026-09-13
> - 원본 위치: `~/Downloads/org-website-handoff.md` (앱 모노레포에는 두지 않는다)
> - 상위 추적 문서: `app-factory/apps/sip-note/docs/store/play-store-account.md` §3 5·6번

---

## 1. 한 줄 목표

**본인 명의 도메인에 개발사(사업자) 소개 정적 사이트를 올리고, Google Play Console 의 웹사이트
인증을 통과시킨다.** 이 인증이 끝나야 Play Console 에 "계정 유형 변경"(개인 → 조직) 옵션이 나타난다.

---

## 2. 배경 — 왜 이 사이트가 필요한가

- 개발자는 Google Play 개인 계정의 "테스터 12명 × 연속 14일 비공개 테스트" 요건을 피하려고
  **조직(Organization) 계정으로 전환**하기로 했다. 조직 계정은 이 요건 대상이 아니다.
- 전환 순서 중 이 사이트가 걸리는 지점:
  1. Play Console **'내 정보'에 공식 조직 웹사이트 URL 입력 → 저장**
  2. **'인증 요청 보내기'** 클릭
  3. 인증 완료 → 비로소 **"계정 유형 변경" 옵션 노출**
  4. 이후 새 결제 프로필(사업자등록번호·D-U-N-S) 연결 → 신원 인증 → 전환 완료
- 많이들 놓치는 관문이다. 사이트가 없으면 1번에서 멈추고 조직 전환 전체가 진행되지 않는다.
- 첫 출시 앱은 **Sip Note** (주류 테이스팅 기록 앱, Android 패키지 `com.infiniteloopfactory.sipnote`).
  완전 로컬 앱이라 광고·결제·서버·계정이 없다.

### 왜 앱 모노레포가 아니라 별도 레포인가 (2026-09-13 결정)

| 이유 | 설명 |
|---|---|
| 인증 대상은 도메인 | 레포 위치는 인증과 무관하다. 본인 명의 도메인만 있으면 된다 |
| Pages 충돌 | 앱 모노레포의 GitHub Pages(`infinite-loop-factory.github.io/app-factory/`)는 앱 웹 빌드가 이미 쓰고 있다. 커스텀 도메인은 레포당 1개라 붙이면 기존 앱 웹 경로가 깨진다 |
| 소유 주체 | 사이트는 사업자등록증·D-U-N-S 의 사업자(=개발자 본인)를 대표한다. 공유 GitHub 조직이 아니라 **본인 계정**이 소유해야 제3자가 수정·삭제해 인증이 풀릴 위험이 없다 |
| 민감 정보 분리 | 사업자등록번호·주소·전화번호를 공유 레포 커밋 기록에 남기지 않는다 |

---

## 3. 확정 필요 값 (작업 시작 전 개발자에게 받을 것)

**추측해서 채우지 말 것.** 값이 없으면 `<자리표시자>` 로 두고 §8 차단 조건을 따른다.

| 키 | 값 | 상태 | 비고 |
|---|---|---|---|
| `DOMAIN` | `<도메인>` | 미정 | 구입 전. 등록자 명의는 본인 |
| `GITHUB_OWNER` | `JAM-PARK` | **확정** (2026-09-13) | 레포 소유 계정 |
| `REPO_NAME` | `org-site` | 임시 (2026-09-13) | 로컬 `~/Desktop/workspace/jam-park/org-site`. 도메인이 정해지면 이름 변경. 원격 레포는 아직 없음 |
| `BIZ_NAME_KO` | `<상호(국문)>` | 미정 | **사업자등록증 표기와 글자 단위로 일치** |
| `BIZ_NAME_EN` | `<상호(영문)>` | 미정 | **D-U-N-S(D&B) 등록 표기와 일치** |
| `REPRESENTATIVE` | `<대표자명>` | 미정 | 표시 여부는 §5.3 결정 |
| `BIZ_REG_NO` | `<사업자등록번호>` | **없음** — 발급 전 | 표시 여부는 §5.3 결정 |
| `ADDRESS` | `<사업장 주소>` | 미정 | 사업자등록증과 일치 |
| `CONTACT_EMAIL` | `<공개 문의 이메일>` | 미정 | 개인 주소와 분리한 **지원용 주소를 새로 만들기로 결정**(2026-08-16). 이 주소가 앱 개인정보처리방침·스토어 연락처와 같아야 한다 |

> **사이트 파일의 자리표시자 표기 (구현 시 결정).** HTML 본문에 `<도메인>`을 그대로 쓰면 태그로 파싱되므로
> 레포 파일에서는 `{{DOMAIN}}`, `{{BIZ_NAME_KO}}` 처럼 **`{{키}}`** 형식을 쓴다. 값이 확정되면 레포 루트에서 치환한다:
>
> ```bash
> grep -rlF '{{DOMAIN}}' --exclude=HANDOFF.md . | xargs sed -i '' 's/{{DOMAIN}}/example.com/g'
> ```
>
> 대표자·사업자등록번호는 각 페이지 푸터에 `발급 후 표시` 주석으로만 들어 있다. 표시하기로 하면 주석을 풀고 치환한다.
>
> **구현 시 선택 항목 (2026-09-13):** 다크 모드 ✅ · Open Graph ✅ · 영문 `/en/` ✅ (홈·앱 소개만. 영문 방침 원문이 없어 영문 페이지는 한국어 방침으로 링크한다)

> **일치 원칙.** 사이트의 상호·주소·연락처 = 사업자등록증 = D-U-N-S = Play 결제 프로필.
> 표기가 하나라도 다르면 Play 쪽 매칭·인증 단계에서 막힌다. 영문 표기는 한 번 정한 값을 복사해서만 쓴다.

---

## 4. 범위

### 포함

| # | 페이지 | 경로 | 필수 | 내용 |
|---|---|---|---|---|
| P1 | 홈(회사 소개) | `/` | ✅ | 상호, 한두 문장 소개, 앱 목록(카드 1개부터), 문의 이메일, 푸터 사업자 정보 |
| P2 | Sip Note 소개 | `/apps/sip-note/` | ✅ | 앱 한 줄 소개, 핵심 기능 3~5개, 스토어 링크(출시 전엔 "출시 준비 중"), 개인정보처리방침 링크 |
| P3 | Sip Note 개인정보처리방침 | `/apps/sip-note/privacy/` | ✅ | §6 원문 이식. Play 스토어 등록 시 이 URL 을 쓴다 |
| P4 | 404 | `/404.html` | ✅ | 홈 링크 |
| P5 | 영문 버전 | `/en/…` | ⬜ 선택 | 영문 스토어 등록 정보용. 1차 인증에는 불필요 |

### 제외 (하지 말 것)

- 애널리틱스·광고·쿠키·외부 폰트·외부 스크립트 — **개인정보처리방침의 "수집 없음"과 모순**되고 제3자 요청이 생긴다
- 문의 폼·뉴스레터·로그인 — 서버·개인정보 처리가 생긴다. 문의는 `mailto:` 로만
- 블로그·CMS·다크 패턴 쿠키 배너
- 앱 모노레포(`app-factory`) 수정

---

## 5. 기술 결정

### 5.1 스택 — **순수 정적 HTML/CSS, 빌드 단계 없음**

- 페이지 4개, 콘텐츠가 거의 안 바뀐다 → 프레임워크·번들러·의존성 업데이트 부담이 이득보다 크다
- GitHub Pages 가 레포 루트를 그대로 서빙 → 배포 = `git push`
- JS 는 필요 없다. 넣더라도 인라인 소량만
- **전환 기준:** 앱이 3개 이상으로 늘어 페이지 복붙이 부담이 되면 Astro(정적 출력)로 옮긴다. 지금은 아니다

### 5.2 호스팅 — GitHub Pages + 커스텀 도메인

- 소스: `main` 브랜치 루트(`/`)
- 레포 루트에 `CNAME` 파일 (내용: `<도메인>` 한 줄)
- 루트에 `.nojekyll` (Jekyll 처리 끔)
- **Enforce HTTPS 켜기** — 인증서 발급까지 수분~최대 24시간

### 5.3 사업자 정보 표시 — 기본값

| 항목 | 기본값 | 근거 |
|---|---|---|
| 상호(국·영문) | **표시** | 인증의 핵심. 조직 실체를 보여 주는 값 |
| 문의 이메일 | **표시** | 공식 연락 수단 |
| 사업장 주소 | **표시** | 결제 프로필·D-U-N-S 와 대조 가능한 실체 정보 |
| 대표자명 · 사업자등록번호 | **발급 후 표시 (권장)** | 판매가 없어 법적 게시 의무는 확인 안 됨. 다만 인증 심사에서 실체 확인에 유리. 개발자가 원치 않으면 생략 |
| 전화번호 | 생략 | 조직 계정은 어차피 Play 스토어에 전화번호가 공개된다. 사이트에 중복 노출할 필요는 없음 |

### 5.4 디자인 제약

- 폰트: 시스템 스택만 — `system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Apple SD Gothic Neo', 'Noto Sans KR', sans-serif`. **웹폰트 CDN 금지**(§4 제외 사유)
- 모바일 우선, 375px 최소 폭에서 가로 스크롤 없음. 본문 16px 이상. 8px 그리드
- WCAG AA: 텍스트 대비 4.5:1 이상, 모든 링크·버튼에 보이는 포커스 스타일, `lang="ko"` 지정
- `prefers-color-scheme` 다크 대응(선택), `prefers-reduced-motion` 존중(애니메이션은 넣지 않는 것이 기본)
- 보라→파랑 그라데이션·블롭 배경 금지. 단색 배경 + 절제된 타이포
- 앱 아이콘이 필요하면 개발자에게 1024×1024 PNG 를 받는다 (앱 레포 `apps/sip-note/src/assets/images/icon.png`)

### 5.5 SEO · 메타

- 모든 페이지에 `<title>`, `<meta name="description">`, `<link rel="canonical" href="https://<도메인>/…">`
- `robots.txt` 는 전체 허용 — **`noindex` 금지**(인증 측이 사이트를 봐야 한다)
- `sitemap.xml` 에 P1~P3
- Open Graph 태그(선택)

---

## 6. P3 개인정보처리방침 — 이식할 원문

앱 레포의 `apps/sip-note/docs/store/privacy-policy-ko.md`(최종 수정일 2026-08-15)를 **내용 변경 없이** HTML 로 옮긴다.
문장을 다듬거나 요약하지 않는다. 구조:

1. 요약 — 어떤 데이터도 수집·전송하지 않음, 계정·서버 없음, 기록은 기기 안에만
2. 기기에 저장되는 정보 (표: 테이스팅 기록 / 사진 / 장소 / 페어링 / 앱 설정)
3. 기기 권한 (표: 카메라 / 사진·미디어 / 위치(사용 중) / 알림 — 목적·거부 시 동작)
4. 위치 정보 — 기기 밖으로 나가지 않음, OS 지오코딩, 백그라운드 수집 없음
5. 지도 — Google Maps SDK 타일 요청, 기록은 전송 안 됨
6. 광고·분석 — SDK 없음
7. 데이터 내보내기와 삭제 — 사용자 실행 시에만 내보내기, 설정 > 데이터 초기화, 앱 삭제 후 복구 불가
8. 아동 — 만 18세 미만 대상 아님
9. 변경 — 최종 수정일 갱신
10. 문의 — `<공개 문의 이메일>` (§3 `CONTACT_EMAIL`)

> 원문 파일을 새 레포로 가져올 때 **최신본인지** 개발자에게 확인한다(원문 §9 문의란이 아직 비어 있다).
> 원문이 바뀌면 사이트도 같은 날 갱신하고 "최종 수정일"을 맞춘다.
> 이식하면서 개발자 이름·개발사 명칭을 넣어야 하면 §3 `BIZ_NAME_KO` 를 쓴다.

---

## 7. 작업 순서

각 단계는 앞 단계 완료 후 진행. **사람 손**이 필요한 단계는 🙋 로 표시했다.

| # | 단계 | 담당 | 완료 기준 |
|---|---|---|---|
| 1 | 🙋 §3 확정 필요 값 수집 (최소: `DOMAIN`, `GITHUB_OWNER`, `REPO_NAME`) | 개발자 | 표에 값이 채워짐 |
| 2 | 🙋 도메인 구입 | 개발자 | 등록자 = 본인, DNS 관리 화면 접근 가능 |
| 3 | 🙋 본인 GitHub 계정에 레포 생성 | 개발자 | `https://github.com/<GITHUB_OWNER>/<REPO_NAME>` |
| 4 | 🙋 GitHub 에서 도메인 사전 인증 — Settings(계정) → Pages → **Add a verified domain** → 안내된 `_github-pages-challenge-<GITHUB_OWNER>` TXT 레코드를 DNS 에 추가 → Verify | 개발자 | "Verified" 표시. 도메인 탈취(subdomain takeover) 방지 |
| 5 | 사이트 골격 작성 — §4 P1~P4, 공통 CSS, `CNAME`, `.nojekyll`, `robots.txt`, `sitemap.xml` | 에이전트 | §8 로컬 검증 통과 |
| 6 | P3 개인정보처리방침 이식 (§6) | 에이전트 | 원문과 문단·표 대조 일치 |
| 7 | 🙋 DNS 레코드 설정 (아래 표) | 개발자 | `dig` 결과가 표와 일치 |
| 8 | 레포 Settings → Pages: Source = `main` / `/ (root)`, Custom domain = `<DOMAIN>`, **Enforce HTTPS** | 개발자 또는 에이전트(`gh` 권한 있으면) | `https://<DOMAIN>/` 200 + 유효 인증서 |
| 9 | §8 배포 검증 | 에이전트 | 전 항목 통과 |
| 10 | 🙋 Play Console '내 정보' → 웹사이트에 `https://<DOMAIN>` 입력·저장 → **'인증 요청 보내기'** | 개발자 | 인증 요청 상태로 바뀜 |
| 11 | 🙋 Play Console 이 요구하는 소유권 확인 수행 | 개발자 | 인증 완료, "계정 유형 변경" 옵션 노출 |

### 7번 DNS 레코드 (GitHub Pages apex + www)

| 이름 | 유형 | 값 |
|---|---|---|
| `@` | A | `185.199.108.153` |
| `@` | A | `185.199.109.153` |
| `@` | A | `185.199.110.153` |
| `@` | A | `185.199.111.153` |
| `@` | AAAA | `2606:50c0:8000::153` |
| `@` | AAAA | `2606:50c0:8001::153` |
| `@` | AAAA | `2606:50c0:8002::153` |
| `@` | AAAA | `2606:50c0:8003::153` |
| `www` | CNAME | `<GITHUB_OWNER>.github.io` |

> 값은 작업 시점에 GitHub 공식 문서
> (https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)
> 로 한 번 더 대조한다.

### 11번 — 소유권 확인 방식 (미확인)

Play Console 웹사이트 인증이 **구체적으로 어떤 방식으로 소유권을 확인하는지는 아직 확인하지 않았다.**
Google Search Console 의 도메인 속성(DNS TXT) 확인을 요구할 가능성이 높으니 **DNS 관리 권한을 계속 쥐고 있을 것.**
HTML 파일 업로드 방식이면 파일을 레포 루트에 커밋하고, 인증 후에도 **지우지 않는다**(지우면 인증이 풀릴 수 있다).
실제 요구 방식은 10번에서 화면에 뜨는 안내를 따르고, §9 결과 보고에 기록한다.

---

## 8. 검증

### 로컬 (5·6단계)

```bash
# 레포 루트에서
python3 -m http.server 8000
# → http://localhost:8000/ , /apps/sip-note/ , /apps/sip-note/privacy/ , /없는경로 확인
```

- [ ] 모든 페이지가 375px / 768px / 1280px 폭에서 가로 스크롤 없이 읽힌다
- [ ] 키보드 Tab 만으로 모든 링크에 도달하고 포커스가 보인다
- [ ] 외부 요청이 0건이다 — DevTools Network 탭에서 `localhost` 외 도메인이 없어야 한다
- [ ] 자리표시자가 남아 있지 않다: `grep -rn '{{' --exclude=HANDOFF.md --exclude-dir=.git . | grep -v '발급 후 표시\|Show after issuance'` 결과 없음 (배포 전 필수)
- [ ] (참고) 자리표시자가 남아 있는 동안 HTML 검사기 오류는 `mailto:{{CONTACT_EMAIL}}` 의 `{` 문자뿐이다. 치환 후 0이 되어야 한다
- [ ] HTML 유효성: https://validator.w3.org/nu/ 에 파일 업로드 → 오류 0
- [ ] 개인정보처리방침이 원문과 문단·표 단위로 일치 (§6)

### 배포 후 (9단계)

```bash
DOMAIN="<도메인>"
dig +short "$DOMAIN" A          # 185.199.108~111.153 네 개
dig +short "$DOMAIN" AAAA       # 2606:50c0:800{0..3}::153
dig +short "www.$DOMAIN" CNAME  # <GITHUB_OWNER>.github.io.
for p in / /apps/sip-note/ /apps/sip-note/privacy/ /robots.txt /sitemap.xml; do
  curl -s -o /dev/null -w "%{http_code} $p\n" "https://$DOMAIN$p"
done                            # 전부 200
curl -s -o /dev/null -w "%{http_code} %{redirect_url}\n" "http://$DOMAIN/"   # 301 → https
curl -s -o /dev/null -w "%{http_code}\n" "https://$DOMAIN/no-such-page"       # 404
```

- [ ] 위 명령 결과가 주석과 일치
- [ ] `https://www.<도메인>/` → apex 로 리다이렉트
- [ ] Lighthouse (모바일) Accessibility ≥ 90, Best Practices ≥ 90
- [ ] 푸터·홈의 상호·주소·이메일이 §3 값과 글자 단위로 일치

### 차단 조건 — 이 경우 멈추고 개발자에게 알린다

- `BIZ_NAME_KO` / `BIZ_NAME_EN` / `CONTACT_EMAIL` 이 비어 있는 채로 **공개 배포**해야 하는 상황
  → 골격(5단계)까지는 진행하되 8번 Pages 공개와 10번 인증 요청은 보류
- 사업자등록 전이라 상호가 확정되지 않음 → 인증 요청(10번) 보류. 상호가 바뀌면 인증을 다시 받아야 할 수 있다
- Play Console 이 요구하는 소유권 확인 방식이 DNS·HTML 파일 어느 쪽도 아님 → 화면 캡처와 함께 보고

---

## 9. 결과 보고 (작업 후 채울 것)

원본 레포의 `play-store-account.md` §3 5·6번 행 갱신에 쓰인다.

| 항목 | 값 |
|---|---|
| 사이트 URL | |
| 레포 URL | |
| 배포 커밋 | |
| HTTPS 인증서 발급 확인일 | |
| GitHub 도메인 사전 인증 | 완료 / 미완료 |
| Play Console 인증 요청일 | |
| Play Console 이 요구한 소유권 확인 방식 | DNS TXT / HTML 파일 / meta 태그 / 기타: |
| Play Console 인증 완료일 | |
| "계정 유형 변경" 옵션 노출 여부 | 노출 / 미노출 |
| 개인정보처리방침 URL (스토어 등록용) | `https://<도메인>/apps/sip-note/privacy/` |
| 미해결 이슈 | |

---

## 10. 참고 링크

| 내용 | 링크 |
|---|---|
| Play — 계정 유형 변경 (웹사이트 인증 선행) | https://support.google.com/googleplay/android-developer/answer/16260648 |
| Play — 조직 계정 D-U-N-S 요건 | https://support.google.com/googleplay/android-developer/answer/13634885 |
| Play — 개인 계정 테스트 요건 (12명/14일) | https://support.google.com/googleplay/android-developer/answer/14151465 |
| GitHub Pages — 커스텀 도메인 | https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site |
| GitHub Pages — 도메인 사전 인증 | https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages |
| Google Search Console — 소유권 확인 | https://support.google.com/webmasters/answer/9008080 |
