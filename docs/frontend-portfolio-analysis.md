# MyFairy 저장소 분석 기반 프론트엔드 포트폴리오 어필 포인트

## 0) 프로젝트 성격 한 줄 요약
- **MyFairy는 실시간 협업형 동화 제작 서비스**이며, 프론트엔드는 단순 화면 구현이 아니라 **실시간 소켓 동기화 + 멀티모달 입력(텍스트/음성/손글씨) + 캔버스 드로잉 + 인증 자동 복구**까지 담당합니다.

---

## 1) 포트폴리오에서 프론트엔드 역할로 강하게 어필할 부분

아래 4가지는 “기능 구현”보다 “문제를 구조적으로 해결했다”는 관점에서 면접관에게 설득력이 높습니다.

### 1-1. 실시간 협업 상태를 STOMP + Zustand로 분리 설계

#### 무엇을 했는가
- `roomStore`에서 STOMP 연결/구독/발행을 중앙 관리하고, 방 생성·참가·시작·퇴장·탈주까지 이벤트를 상태로 통합했습니다.
- `/topic/room/*`, `/topic/room/start/*`, `/topic/room/escape/*` 채널을 각각 다루며 협업 상태를 즉시 반영합니다.

#### 왜 중요한가
- 실시간 협업 서비스에서 가장 어려운 부분은 “페이지가 아니라 **상태를 기준으로 동기화**”하는 것입니다.
- 이 구조는 UI 컴포넌트가 소켓 구현 세부사항을 몰라도 되도록 분리해 유지보수성/확장성을 높입니다.

#### 면접용 한 줄
> “실시간 기능을 컴포넌트에 흩뿌리지 않고 `roomStore`에 모아, 이벤트 채널·상태 전이를 한 곳에서 제어하도록 설계했습니다.”

---

### 1-2. 제작 플로우 상태를 별도 스토어로 분리해 도메인 결합도 축소

#### 무엇을 했는가
- `playStore`에서 키워드 입력(타이핑/음성/손글씨), 최종 제출, 그림 제출, 완성 이벤트를 담당합니다.
- `roomStore`(룸 실시간 라이프사이클)와 `playStore`(동화 제작 도메인)를 분리했습니다.

#### 왜 중요한가
- 실무에서 소켓 수명주기와 비즈니스 플로우 로직이 섞이면 디버깅이 어렵습니다.
- 역할 분리는 테스트/리팩토링 시 영향 범위를 줄이고, 기능 추가 시 안정성을 높입니다.

#### 면접용 한 줄
> “소켓 연결 책임과 제작 액션 책임을 스토어 단위로 나눠, 상태 복잡도를 도메인별로 분해했습니다.”

---

### 1-3. 드로잉 캔버스 + 실시간 스트리밍(라이브 트랙) 연동

#### 무엇을 했는가
- `DrawingBoard`를 `forwardRef + useImperativeHandle`로 구성해 `getPNGFile`, `clearCanvas`, `getCanvas` 같은 제어 API를 외부로 노출했습니다.
- `TaleSentenceDrawing`에서 캔버스를 `captureStream(30)`으로 스트림화하고, LiveKit 트랙으로 퍼블리시합니다.
- 제한시간 기반 자동 제출/단계 전환까지 포함해 UX 흐름을 완성했습니다.

#### 왜 중요한가
- 단순 Canvas 그림판이 아니라, **그림 데이터를 파일 업로드와 실시간 영상 트랙으로 이중 활용**한 점이 고급 프론트엔드 역량으로 보입니다.
- 사용자 상호작용(타이머, 경고음, 자동 제출)을 일관된 상태 전이로 연결한 점이 제품 관점에서 강점입니다.

#### 면접용 한 줄
> “Canvas를 UI 요소가 아니라 데이터 소스로 다뤄, 파일 제출과 실시간 전송에 동시에 활용했습니다.”

---

### 1-4. 인증 만료를 사용자 체감 없이 복구하는 axios 인터셉터 설계

#### 무엇을 했는가
- 요청 인터셉터에서 Access Token 부착, 응답 인터셉터에서 401 발생 시 refresh 후 원요청 재시도 로직을 구현했습니다.
- 동시에 여러 요청이 401을 맞아도 `isRefreshing + subscriber queue`로 refresh 중복 요청을 막습니다.

#### 왜 중요한가
- 실제 서비스에서 흔한 장애는 “토큰 만료 시 연쇄 실패”입니다.
- 이 구조는 서버 부하와 프론트 오류 폭발을 줄이고, 사용자 입장에서는 끊김 없는 경험을 제공합니다.

#### 면접용 한 줄
> “401 동시다발 상황에서 단일 재발급만 발생하도록 동시성 제어를 넣어, 인증 복구 안정성을 확보했습니다.”

---

## 2) 트러블슈팅 1개 (근거 코드 + 워크플로우 포함)

아래 주제는 코드 근거가 명확하고, 프론트엔드 문제 해결 역량을 보여주기 좋습니다.

## [트러블슈팅] 동시다발 401 발생 시 Refresh 폭주로 인한 요청 실패

### 상황(Situation)
- Access Token 만료 시점에 API 요청이 여러 개 동시에 발생하면, 각 요청이 모두 401을 받아 refresh API를 중복 호출하는 문제가 발생할 수 있었습니다.
- 이 경우 refresh 경합으로 실패율이 증가하고, 일부 요청은 재시도 타이밍이 꼬여 로그아웃으로 이어질 위험이 있었습니다.

### 과제(Task)
- refresh 요청을 **한 번만** 보내고, 나머지 요청은 새 토큰을 기다렸다가 정상 재시도되도록 만들기.
- 사용자에게는 새로고침/재로그인 없이 자연스럽게 복구되는 인증 UX 제공.

### 실행(Action)
1. 전역 플래그 `isRefreshing`을 도입해 refresh 진행 중복을 차단.
2. refresh 진행 중 들어온 요청은 subscriber queue에 콜백으로 등록.
3. refresh 성공 시 `onRefreshed(newToken)`으로 큐에 쌓인 요청을 일괄 재개.
4. 응답 인터셉터에서 401인 원요청에 `_retry` 플래그를 두어 무한 루프 방지.
5. refresh 실패 또는 403이면 즉시 logout 처리해 비정상 상태를 정리.

### 결과(Result)
- 토큰 만료 타이밍의 연쇄 오류를 크게 줄이고, 인증 복구를 일관되게 처리할 수 있는 구조를 확보했습니다.
- 사용자는 “일시적 인증 만료”를 거의 인지하지 않고 계속 기능을 사용할 수 있습니다.
- 백엔드 관점에서도 refresh API 중복 호출이 줄어 부하 완화 효과가 있습니다.

### 트러블슈팅 워크플로우 (Mermaid)
```mermaid
flowchart TD
  A[API 요청 발생] --> B{AccessToken 만료?}
  B -- 아니오 --> C[Authorization 헤더 부착 후 요청]
  B -- 예 --> D[요청은 전송되나 401 가능]
  C --> E{응답 상태}
  D --> E

  E -- 200/SU --> Z[정상 종료]
  E -- 401 --> F{_retry 여부}
  F -- 이미 true --> X[중복 재시도 방지 후 실패 반환]
  F -- false --> G[_retry=true 설정]

  G --> H{isRefreshing?}
  H -- true --> I[refreshSubcribers 큐에 콜백 등록]
  I --> J[새 토큰 발급 완료 대기]
  J --> K[토큰 수신 후 원요청 재실행]
  K --> Z

  H -- false --> L[isRefreshing=true]
  L --> M[/auth/refresh 호출]
  M --> N{refresh 성공?}
  N -- 성공 --> O[onRefreshed로 큐 일괄 재개]
  O --> P[store.accessToken 갱신]
  P --> K

  N -- 실패 --> Q[logout 및 인증상태 초기화]
  Q --> R[에러 반환]

  E -- 403 --> S[logout 처리]
  S --> R
```

### 근거 코드 (핵심 포인트)

#### 1) 중복 refresh 차단 플래그/큐 선언
- `isRefreshing`으로 refresh 단일화 여부를 제어합니다.
- `refreshSubcribers` 배열로 대기 요청을 큐잉합니다.

```js
let isRefreshing = false;
let refreshSubcribers = [];
```

#### 2) refresh 완료 시 큐에 쌓인 요청 재개
- refresh 성공 후 `onRefreshed(newToken)`에서 대기 콜백을 모두 실행합니다.

```js
const onRefreshed = (newToken) => {
  refreshSubcribers.map((callback) => callback(newToken));
  refreshSubcribers.length = 0;
};
```

#### 3) refresh 진행 중 요청은 새 refresh를 보내지 않고 대기
- 이미 refresh 중이면 Promise를 반환하고 subscriber를 등록해 토큰을 기다립니다.

```js
if (isRefreshing) {
  return new Promise((resolve) => {
    addRefreshSubscriber((newToken) => {
      resolve(newToken);
    });
  });
}
```

#### 4) 인터셉터에서 401 재시도/403 로그아웃 처리
- `_retry` 플래그로 재시도 루프를 방지합니다.
- refresh 성공 시 원요청 헤더를 갱신해 재실행합니다.

```js
if (error.response?.status === 401 && !originalRequest._retry) {
  originalRequest._retry = true;
  await refreshAccessToken();
  originalRequest.headers['Authorization'] =
    `Bearer ${userStore.getState().accessToken}`;
  return api(originalRequest);
}

if (error.response?.status === 403) {
  logout();
}
```

### 면접에서 강조할 포인트
- “단순 인터셉터 구현”이 아니라 **동시성 문제를 큐 기반으로 제어**했다는 점.
- 실패 시나리오(401/403/refresh 실패)까지 설계한 **운영 관점의 견고성**.

---

## 3) 포트폴리오 문서에 넣기 좋은 정리 포맷

### A. 핵심 기술 스택
- React + Vite
- Zustand(스토어 분리: room/play/user)
- STOMP/SockJS(WebSocket)
- Axios 인터셉터(자동 재발급)
- LiveKit + Canvas 기반 실시간 드로잉

### B. 내가 기여한 프론트엔드 가치 (예시 문장)
- “실시간 협업 이벤트를 상태 중심으로 설계해 동기화 안정성을 높였습니다.”
- “멀티모달 입력과 드로잉 워크플로우를 통합해 사용자 참여 경험을 강화했습니다.”
- “인증 만료 복구를 자동화하고 동시성 제어를 적용해 운영 안정성을 개선했습니다.”

### C. 성과 지표(추후 수치화 추천)
- 인증 실패 재시도율, refresh 중복 호출 감소량
- 방 입장/시작/제작 완료 단계별 이탈률
- 제작 완료까지 평균 소요시간, 드로잉 제출 성공률

---

## 4) 참고 코드 위치
- 실시간 룸 소켓/상태 관리: `frontend/src/store/roomStore.js`
- 제작 도메인 상태/제출 로직: `frontend/src/store/tale/playStore.js`
- 인증 API: `frontend/src/apis/auth/userAxios.js`
- 토큰 저장/재발급/인터셉터: `frontend/src/store/userStore.js`
- 드로잉 캔버스 컴포넌트: `frontend/src/components/Common/DrawingBoard.jsx`
- 드로잉 페이지/타이머/캔버스 트랙 퍼블리시: `frontend/src/pages/Room/TaleSentenceDrawing.jsx`
