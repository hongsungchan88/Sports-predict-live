> **이 저장소는 4인 팀 프로젝트 [Clutch-team-project/Sports-predict-live](https://github.com/Clutch-team-project/Sports-predict-live)의 포크입니다.**
> 원본 저장소에 README가 없어, **홍성찬([@hongsungchan88](https://github.com/hongsungchan88))이 담당한 부분**을 정리해 두었습니다.
> 코드는 원본과 동일하고, 아래 설명은 담당 파트 기준으로 작성했습니다.

**포트폴리오** · https://hongsungchan88.github.io

---

# SPORTS PREDICT LIVE

야구·축구·LoL 경기를 실시간으로 중계하고, AI 기반 승부예측을 제공하는
스포츠 통합 플랫폼입니다.

> 팀 프로젝트 (4인) · 2026.06.01 ~ 2026.06.29
> **담당**: 회원 · 경기 일정 · 순위/기록 · AI 예측 파트 (백엔드)

## 기술 스택 (팀 전체)

| 구분 | 사용 기술 |
|---|---|
| Language | Java 21 |
| Framework | Spring Boot 3.5, Spring Data JPA, Spring Security, QueryDSL |
| Realtime | WebSocket |
| Auth | OAuth2 (Kakao · Google), JWT |
| AI | OpenAI API |
| Infra | AWS EC2 |
| Crawler | Python (KBO · K리그 · LCK) |

## 규모

| 항목 | 수치 |
|---|---|
| 세부 기능 정의 | 11개 대분류 · 91건 |
| 테스트케이스 수행 | 17개 영역 · 193건 |

## 제가 구현한 부분

- Spring Security · JWT 인증, 카카오 · 구글 OAuth2 소셜 로그인, 회원가입 이메일 인증
- 경기 일정 · 순위 · 기록 API
- OpenAI API 연동 승/무/패 확률 예측과 근거 제공 (응답 스키마 고정, 경기당 DB 캐싱)
- 소셜 로그인 후 토큰 전달 위치를 URL 쿼리스트링에서 fragment(#)로 변경
- KBO 크롤러 일부 (선발 · 승패 투수 수집, 컬럼 매핑)
- 분석 단계 기능 정의서 91건 작성

실시간 중계(WebSocket), AI 댓글·게시글 필터링, 게시판(QueryDSL)은 다른 팀원이 담당했습니다.

## 트러블슈팅

### 1. 배포 환경에서만 이메일 인증이 실패했습니다

로컬에서는 정상 동작하던 회원가입 이메일 인증이 AWS EC2 배포 후에만 실패했습니다.
코드가 동일했기 때문에 애플리케이션 로그만으로는 원인이 보이지 않았습니다.

**해결** — 로컬과 배포 환경의 차이를 네트워크 · 계정 · 포트 순으로 하나씩 비교하다가,
AWS가 기본적으로 특정 SMTP 포트의 아웃바운드를 차단한다는 것을 확인했습니다.
허용 포트로 변경해 재배포한 뒤 메일 발송을 검증했습니다.
이때부터 인프라 설정도 디버깅 대상에 넣게 됐습니다.

### 2. 소셜 로그인 토큰이 URL 쿼리스트링에 노출됐습니다

소셜 로그인 후 JWT를 URL 쿼리스트링으로 프론트에 넘기고 있었습니다.
쿼리스트링은 서버 로그 · 브라우저 기록 · Referer 헤더에 남을 수 있습니다.

**해결** — 토큰을 URL fragment(#)로 전달하도록 바꿔 서버 요청 URL에 토큰이 실리지 않게 했습니다.

### 3. AI 응답 형식이 불안정했습니다

GPT 응답에 마크다운과 설명 텍스트가 섞여 들어와 JSON 파싱이 깨졌습니다.

**해결** — `response_format: json_object` 를 강제하고 시스템 프롬프트에 스키마를
명시해, 응답이 항상 파싱 가능한 형태로 오도록 고정했습니다.

### 4. AI API 호출 비용을 통제했습니다

예측 요청이 들어올 때마다 API를 호출하면 같은 경기에 대해 중복 호출이 발생했습니다.
별도의 rate limiter를 붙이는 대신, 경기당 예측 결과를 DB에 캐싱해 재호출을 막는
구조로 경기당 1회 호출로 제한했습니다.

## 실행 방법

```bash
# 1. 환경 변수 설정
export DB_URL=jdbc:mysql://localhost:3306/sports
export DB_USERNAME=root
export DB_PASSWORD=여기에_비밀번호
export OPENAI_API_KEY=여기에_키
export KAKAO_CLIENT_ID=...
export GOOGLE_CLIENT_ID=...

# 2. 실행
./gradlew bootRun
```
