# MyFairy 저장소 분석 기반 포트폴리오 정리

## 문서 목적
이 문서는 **프론트엔드 포지션 지원 시 이 프로젝트에서 무엇을 어필하면 좋은지**를 저장소 코드 기준으로 정리한 자료입니다.  
특히 면접/포트폴리오에서 바로 활용할 수 있도록,

1. 프론트엔드 역할로 강조할 포인트
2. 트러블슈팅 사례 1개(문제-원인-해결-성과)

를 중심으로 구성했습니다.

---

## 1) 프론트엔드 역할로 어필할 핵심 포인트

### A. "실시간 협업" 설계/구현 역량 (WebSocket + 상태관리)

**어필 문장(한 줄):**  
`SockJS + STOMP 기반 실시간 룸 협업 기능을 Zustand 스토어로 모듈화해 방 생성/입장/시작/이탈 시나리오를 안정적으로 제어했습니다.`

**근거 코드**
- 소켓 연결/헤더 인증/메인 채널 구독: `frontend/src/store/roomStore.js`의 `connectRoom`, `subscribeMain`
- 룸 이벤트 구독 및 액션 분리: `joinRoom`에서 room/start/leave/escape 채널 처리
- 제작 단계 구독 분리: `frontend/src/store/tale/playStore.js`의 `setSubscribeTale` (`/topic/tale/{roomId}`, `/finish`)

**포트폴리오에 쓰기 좋은 포인트**
- 방 단위 실시간 상태와 제작 단계 상태를 store 수준에서 책임 분리(결합도 감소)
- 백엔드 브로드캐스트를 프론트에서 멀티/싱글 모드로 재분배
- 탈주/종료 같은 예외 이벤트도 실시간 플로우에 포함

---

### B. "인증 UX 안정화" 역량 (JWT 자동 재발급 + 동시 요청 제어)

**어필 문장(한 줄):**  
`Axios 인터셉터와 토큰 스토어를 결합해 만료 토큰 자동 재발급 플로우를 구축하고, 동시 401 발생 시 refresh 중복 호출을 큐잉으로 제어했습니다.`

**근거 코드**
- 인증 상태 persist: `frontend/src/store/userStore.js` (`persist`, `sessionStorage`)
- 만료 판별 및 refresh 함수: `decodeJWT`, `isTokenExpired`, `refreshAccessToken`
- 동시성 제어: `isRefreshing`, `refreshSubcribers`, `onRefreshed`
- 전역 요청/응답 인터셉터: 같은 파일 하단 `api.interceptors.request/response`

**포트폴리오에 쓰기 좋은 포인트**
- 사용자 입장에서 "다시 로그인" 마찰을 최소화
- API 다발 호출 상황에서도 refresh storm 방지
- 인증 로직을 컴포넌트가 아닌 인프라 레이어(store/interceptor)에 응집

---

### C. "복합 입력 UX" 구현 역량 (타이핑/음성/손글씨)

**어필 문장(한 줄):**  
`동화 키워드 입력을 타이핑·음성·손글씨 3가지 모드로 구현하고, 동일한 제작 플로우에 자연스럽게 통합했습니다.`

**근거 코드**
- 입력 모드별 제출 함수: `frontend/src/store/tale/playStore.js`
  - `submitTyping`
  - `submitVoice` (webm → wav 변환 + multipart)
  - `submitHandWrite`
- 모드별 API 엔드포인트 매핑: `frontend/src/apis/tale/taleAxios.js`

**포트폴리오에 쓰기 좋은 포인트**
- 사용자 환경/선호도(텍스트, 마이크, 그림)에 맞는 대체 입력 수단 제공
- 입력 방식이 달라도 최종 상태 전이는 동일하게 유지(도메인 모델 일관성)
- 실시간 게임형 UX에서 접근성/몰입감 개선

---

### D. "캔버스 기반 인터랙션" 구현 역량 (그림 그리기/내보내기/스트리밍)

**어필 문장(한 줄):**  
`Canvas 드로잉 컴포넌트를 직접 설계해 마우스/터치 입력, PNG 파일 추출, 완료 상태 제어까지 구현했습니다.`

**근거 코드**
- 캔버스 엔진: `frontend/src/components/Common/DrawingBoard.jsx`
  - 마우스/터치 이벤트 처리
  - `getPNGFile`, `clearCanvas`, `completeDrawing` 노출(`forwardRef`)
- 드로잉 단계 페이지: `frontend/src/pages/Room/TaleSentenceDrawing.jsx`
  - 제한 시간/자동 제출
  - 단계 진행 및 이전 그림 관리
  - LiveKit 캔버스 트랙 publish

**포트폴리오에 쓰기 좋은 포인트**
- 단순 폼 UI를 넘어 고난도 인터랙션 개발 경험
- 멀티미디어(캔버스, 오디오, 화상)와 실시간 협업 기능의 결합
- 사용자 행동(시간 초과/이탈)까지 고려한 플로우 완성

---

## 2) 트러블슈팅으로 어필할 1개 사례 (추천)

아래 사례를 포트폴리오의 "문제 해결" 섹션에 그대로 활용하는 것을 추천합니다.

### 사례: "그림 그리기 단계 중 사용자 이탈 시 데이터 유실" 대응

#### 문제 상황 (Problem)
4인 협업 제작에서 누군가 그림 단계 중 이탈하면, 남은 인원이 계속 진행해도 **완성 조건 미충족으로 세션이 깨지거나 산출물이 유실**될 위험이 있었습니다.

#### 원인 분석 (Cause)
- 제작 초반 이탈과 그림 단계 이탈을 동일하게 처리하면,
  - 초반에는 즉시 세션 종료가 맞지만
  - 그림 단계에서는 "지금 그리고 있던 결과"를 제출해야 데이터 일관성을 유지할 수 있음
- 즉, 이탈 처리 정책이 단계별로 달라야 했음

#### 해결 방법 (Action)
1. **이탈 이벤트를 단계별 채널로 분리**
   - `escape/before` vs `escape/after`로 구분
   - 룸 스토어에서 각각 구독하도록 구현 (`roomStore.js`)
2. **그림 단계 이탈 시 자동 보정 제출**
   - `TaleSentenceDrawing.jsx`에서 `isEscape` 감지 시 `handleConfirm()` 호출
   - 현재 캔버스를 PNG로 추출해 즉시 제출
3. **흐름 문서화**
   - `docs/tale-room-communication-interview.md`에 before/after 의도와 시퀀스를 명확히 정리

#### 결과 (Result)
- 이탈이 발생해도 "그림 단계 산출물"이 최대한 보존되어 방 전체 실패율 감소
- 예외 상황이 클라이언트 로직에 명시적으로 반영되어 재현/디버깅 용이
- 협업형 서비스에서 필요한 "실시간 예외 제어" 역량을 포트폴리오에서 강하게 어필 가능

#### 면접용 30초 요약
`협업 동화 제작 중 이탈 이벤트를 before/after로 분리해 단계별 실패 정책을 다르게 설계했습니다. 그림 단계 이탈은 클라이언트에서 자동 제출로 보정해 데이터 유실을 줄였고, 결과적으로 실시간 협업 플로우의 안정성을 높였습니다.`

---

## 3) 포트폴리오 문구 템플릿 (복붙용)

### 3-1. 역할 소개 문구
- `React + Zustand 기반으로 실시간 협업 동화 제작 클라이언트를 개발했습니다.`
- `SockJS/STOMP, JWT 자동 재발급, Canvas 드로잉, LiveKit 연동까지 프론트엔드 도메인을 주도했습니다.`

### 3-2. 기술적 기여 문구
- `실시간 방 상태(입장/시작/이탈)와 제작 상태(키워드/그림/완료)를 스토어로 분리하여 유지보수성을 개선했습니다.`
- `동시 401 요청에서 refresh 중복 호출을 큐로 제어해 인증 안정성을 확보했습니다.`
- `그림 단계 이탈 시 자동 제출 보정 로직을 도입해 협업 데이터 유실을 완화했습니다.`

### 3-3. 회고/개선 문구
- `예외 흐름(이탈, 타임아웃, 재접속)을 일반 흐름과 같은 수준으로 설계해야 실시간 서비스 품질이 올라간다는 점을 학습했습니다.`

---

## 4) 참고한 주요 파일

- `frontend/src/store/roomStore.js`
- `frontend/src/store/tale/playStore.js`
- `frontend/src/store/userStore.js`
- `frontend/src/components/Common/DrawingBoard.jsx`
- `frontend/src/pages/Room/TaleSentenceDrawing.jsx`
- `docs/tale-room-communication-interview.md`
- `docs/jwt-authentication-analysis-interview.md`

