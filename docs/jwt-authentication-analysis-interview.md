# MyFairy JWT 기반 로그인/인증 체계 분석 (면접·포트폴리오용)

## 1) 한눈에 보는 인증 아키텍처

```mermaid
flowchart LR
  U[사용자 브라우저\nReact + Zustand] -->|아이디/비번 또는 카카오 code| A[/api/auth/login\n/api/auth/kakao/callback/]
  A --> S[Spring AuthService/KakaoService]
  S --> J[JwtUtil\nAccess/Refresh 발급]
  S --> R[(Redis)\nrefresh:{loginId} 저장]
  S --> U

  U -->|Authorization: Bearer accessToken| F[JwtAuthenticationFilter]
  F --> C[SecurityContext 인증 주입]
  C --> API[보호된 API Controller]

  U -->|401 응답 시| RE[/api/auth/refresh\nrefreshToken 전달/]
  RE --> V[RefreshToken 검증\nJWT 서명+만료 + Redis 일치]
  V --> NA[새 AccessToken 발급]
  NA --> U

  U -->|웹소켓 CONNECT + Authorization 헤더| WS[FilterChannelInterceptor]
  WS --> WSC[STOMP 세션 사용자 설정]
```

---

## 2) 로그인 시나리오

### 2-1. 일반 로그인 (ID/PW)
1. 프론트엔드에서 `POST /api/auth/login` 호출.
2. 백엔드는 `AuthService.login()`에서
   - `loginId`로 사용자 조회,
   - 비밀번호 검증,
   - Access/Refresh JWT 발급,
   - Refresh Token을 Redis에 저장.
3. 응답으로 `member` + `tokens(accessToken, refreshToken)`를 전달.
4. 프론트엔드는 Zustand store/sessionStorage에 토큰과 사용자 정보를 저장.

### 2-2. 카카오 로그인
1. 프론트는 카카오 인가 코드(`code`)를 받아 `GET /api/auth/kakao/callback` 호출.
2. 백엔드는 카카오 토큰/프로필 API를 호출해 사용자 식별.
3. 기존 회원이면 로그인, 없으면 자동 회원 생성 후 JWT(Access/Refresh) 발급.
4. Refresh Token은 Redis에 저장하고, 프론트로 동일한 토큰 구조를 반환.

---

## 3) 서비스 전반 인증(인가) 동작

### 3-1. HTTP API 인증
- `WebSecurityConfig`는 세션을 `STATELESS`로 두고 JWT 필터를 등록.
- 일부 엔드포인트(`/api/auth/login`, `/api/auth/refresh`, `/api/auth/kakao/callback` 등)는 `permitAll`.
- 그 외 대부분 엔드포인트는 인증 필요(`anyRequest().authenticated()`).

### 3-2. JWT 검증 필터
- `JwtAuthenticationFilter`가 요청의 `Authorization: Bearer <token>`을 파싱.
- `JwtUtil.extractUsername()`으로 subject(loginId)를 추출.
- `UserDetailsService`로 사용자 로드 후 `validateToken()` 검증.
- 성공 시 `SecurityContext`에 인증 객체를 넣어 컨트롤러 `Authentication` 파라미터에서 사용 가능.

### 3-3. WebSocket/STOMP 인증
- `/ws` 엔드포인트는 연결 자체는 열려 있으나, CONNECT 프레임에서 `Authorization` 헤더를 검사.
- `FilterChannelInterceptor`가 JWT 유효성 검사 후 STOMP 세션 사용자 정보를 설정.

---

## 4) 토큰 재발급/로그아웃 정책

### 4-1. 재발급
- 프론트 axios interceptor가 401 발생 시 1회 재시도 플로우 실행.
- `POST /api/auth/refresh`에 refreshToken 전송.
- 백엔드는
  1. Refresh JWT 자체 유효성(서명/만료) 확인,
  2. Redis 저장값과 일치 여부 확인,
  3. 새 Access Token만 발급.
- 프론트는 새 Access Token으로 원요청을 재실행.

### 4-2. 로그아웃
- `POST /api/auth/logout` 호출 시 Authorization 헤더에서 loginId 추출.
- Redis의 `refresh:{loginId}` 키를 삭제하여 재발급 경로 차단.
- Access Token은 stateless 특성상 즉시 서버 폐기 목록 처리 없이 만료까지 유효.

---

## 5) 프론트엔드 인증 상태관리 핵심

- Zustand persist를 `sessionStorage`에 연결해 브라우저 세션 단위 로그인 유지.
- 요청 인터셉터:
  - `/auth/refresh` 요청은 Authorization 헤더 미첨부,
  - 그 외 요청은 만료되지 않은 Access Token만 Bearer로 첨부.
- 응답 인터셉터:
  - 401이면 refresh 수행 후 원요청 재시도,
  - refresh 실패 또는 403이면 logout 처리.
- 동시다발 401 상황에서 `isRefreshing + queue(subscribers)`로 중복 재발급 방지.

---

## 6) 보안 설계 포인트 (면접에서 강조하기 좋은 부분)

1. **Stateless 인증 + 서버 저장형 Refresh 조합**
   - Access는 빠른 검증, Refresh는 Redis 대조로 탈취 위험 완화.
2. **필터 기반 횡단 인증 처리**
   - 컨트롤러마다 중복 검증 없이 Security Filter에서 일괄 처리.
3. **재발급 경쟁 상태 제어(프론트)**
   - 다중 API 동시 401 발생 시 refresh storm 방지.
4. **실시간 채널 인증 확장**
   - HTTP뿐 아니라 STOMP CONNECT 단계에도 JWT 검증 적용.

---

## 7) 개선 여지/리스크 (포트폴리오에 ‘다음 단계’로 제시 추천)

1. **Access Token 블랙리스트(선택)**
   - 현재는 로그아웃 시 Refresh만 제거하므로 Access는 만료 전까지 유효.
2. **Refresh Rotation**
   - 재발급 시 Refresh도 함께 재발급/교체하면 재사용 공격 대응 강화.
3. **토큰 저장 위치 보안 강화**
   - 현재 sessionStorage 기반이므로 XSS 하에서 노출 위험 존재.
   - HttpOnly Secure SameSite 쿠키 전략을 대안으로 검토 가능.
4. **WebSocket 엔드포인트 접근정책 정교화**
   - `/ws/** permitAll` + CONNECT 검증 구조를 유지하되, 운영 정책 문서화 필요.

---

## 8) 면접 1분 요약 스크립트

> MyFairy는 Spring Security의 Stateless JWT 구조를 사용합니다. 로그인 시 Access/Refresh 토큰을 발급하고, Refresh 토큰은 Redis에 저장해 서버가 세션 유사 통제를 합니다. 일반 API는 `JwtAuthenticationFilter`에서 Bearer 토큰을 검증해 SecurityContext에 인증을 주입하고, 만료 시 프론트 axios 인터셉터가 `/auth/refresh`로 자동 재발급을 수행합니다. 또한 WebSocket STOMP CONNECT에도 JWT 검증 인터셉터를 적용해 실시간 채널까지 동일한 인증 체계를 확장했습니다. 결과적으로 응답성은 유지하면서도 Refresh 검증으로 보안을 보완한 구조입니다.

---

## 9) 실제 코드 기준 근거 포인트

- 인증/인가 정책: `WebSecurityConfig`
- JWT 발급/검증: `JwtUtil`
- HTTP JWT 필터: `JwtAuthenticationFilter`
- 로그인/로그아웃/재발급 API: `AuthController`, `AuthService`
- 카카오 로그인 + JWT 브릿지: `KakaoService`
- Refresh Redis 저장소: `RefreshTokenService`
- 프론트 토큰 저장/자동재발급: `frontend/src/store/userStore.js`
- 프론트 인증 API 호출: `frontend/src/apis/auth/userAxios.js`
- WebSocket CONNECT 인증: `FilterChannelInterceptor`, `WebSocketConfig`

