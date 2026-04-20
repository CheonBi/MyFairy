# 동화 제작(4인 방) 프론트-백엔드 통신 구조 분석

이 문서는 **4인 방 입장 → 키워드 각색 → 동화 생성 → 손그림 제출 → AI 재탄생** 흐름에서,
프론트(React/Zustand/SockJS+STOMP)와 백엔드(SpringBoot/STOMP/Redis/MySQL, Python AI)가
어떻게 통신하는지 기술 면접용으로 정리한 문서입니다.

---

## 1) 아키텍처 한눈에 보기

- **실시간 제어/상태 동기화(방, 시작, 단계 전환 알림)**: STOMP over SockJS
  - 프론트 publish: `/app/**`
  - 프론트 subscribe: `/topic/**`, `/active/**`
- **파일/AI 연동/조회성 API**: HTTP REST (multipart/json)
- **상태 저장 전략**
  - **Redis**: 방 대기 상태(`tale-{roomId}`, `tale-roomList`) + 제작 중 임시 페이지(`tale_member-{id}`)
  - **MySQL**: 영구 데이터(Tale, TaleMember, BaseTale 등)
  - **S3**: 음성/손그림/AI그림 파일 저장 URL

핵심 포인트: 방/단계 전환은 소켓, 무거운 데이터(파일/AI)는 REST + 비동기 콜백으로 분리한 구조.

---

## 2) WebSocket(STOMP) 채널/역할

### 백엔드 STOMP 설정
- 엔드포인트: `/ws` (+ SockJS)
- 브로커 prefix: `/topic`, `/queue`, `/active`
- 앱 destination prefix: `/app`

즉, 프론트는 `/app/...`로 보내고 백엔드는 `/topic/...`으로 브로드캐스트합니다.

### 룸 관련 실시간 이벤트

#### 프론트 publish
- 방 생성: `/app/room/create`
- 방 참가: `/app/room/join/{roomId}`
- 방 나가기: `/app/room/leave/{roomId}`
- 게임 시작: `/app/room/start/{roomId}`
- 탈주 처리: `/app/room/escape/before/{roomId}`, `/app/room/escape/after/{roomId}/{memberId}`

#### 프론트 subscribe
- 생성 결과: `/topic/rooms`
- 방 상태/참여자: `/topic/room/{roomId}`
- 시작 payload: `/topic/room/start/{roomId}`
- 퇴장 이벤트: `/topic/room/leave/{roomId}`
- 탈주 이벤트: `/topic/room/escape/before/{roomId}`, `/topic/room/escape/after/{roomId}/{memberId}`

### `escape before` / `escape after`가 나뉜 이유

코드 기준으로 탈주 이벤트는 **게임 단계 경계(그림 그리기 전/후)** 를 나누기 위해 분리되어 있습니다.

- **before**: `taleStart`, `taleKeyword` 단계에서 이탈  
  - 아직 그림 제출 단계에 진입하지 않았으므로, 방을 **즉시 break**하는 시나리오에 가깝습니다.
  - 백엔드도 `/app/room/escape/before/{roomId}` 수신 시 `taleService.breakRoom(roomId)`를 호출해
    room/tale_member redis 및 DB를 정리합니다.
- **after**: `taleSentenceDrawing` 단계(그림 그리는 중)에서 이탈  
  - 이탈 시점에 이미 그리기 단계가 시작되었기 때문에, 프론트는 `/app/room/escape/after/{roomId}/{memberId}`를 발행하고
    그리기 페이지에서는 `isEscape` 감지 시 현재 캔버스를 `handleConfirm()`으로 제출하도록 처리합니다.
  - 즉 “무효화/즉시 방파기(before)”와 “진행중 산출물 보존 후 종료(after)”를 구분하려는 의도입니다.

면접에서 한 줄 요약:  
**before는 제작 초반 이탈(세션 파기), after는 그림 단계 이탈(제출 보정 후 종료)로 단계별 실패 처리 정책이 다르기 때문**.

### 동화 제작 단계 이벤트
- 동화 문장(누가 어떤 문장 그릴지) 전달: `/topic/tale/{roomId}`
- 전체 제작 종료 알림: `/topic/tale/{roomId}/finish`

---

## 3) 단계별 상세 시퀀스 (면접에서 말하기 좋은 버전)

## A. 4인 방 입장/준비

1. 프론트가 SockJS(`/ws`) 연결 후 STOMP 활성화.
2. 방 생성 시 `/app/room/create` publish → 백엔드 `RoomService.makeRoom`:
   - Tale row 생성(MySQL)
   - Room 객체를 Redis `tale-{roomId}` 저장
   - room list를 Redis `tale-roomList`에 반영
3. 방 참가 시 `/app/room/join/{roomId}` publish → `RoomService.joinRoom`:
   - Redis room participants 갱신
   - 인원수 반영, full 처리
4. 시작 시 `/app/room/start/{roomId}` publish → `TaleService.startMakingTale`:
   - BaseTale 키워드 문장 4개를 참가자와 매핑
   - TaleMember 4개 생성(MySQL)
   - 제작용 DTO를 Redis(`tale_member-{id}`)로 이동
   - 대기 room list 정리
   - 결과를 `/topic/room/start/{roomId}`로 전달

면접 포인트: “방 로비는 Redis 중심, 실제 제작 시작 시점에 MySQL+Redis 혼합 모델로 전환”

## B. 키워드 각색 (타이핑/음성/손글씨)

1. 입력 방식별 API 호출
   - 타이핑: `POST /api/tale/keyword/typing`
   - 음성: `POST /api/tale/keyword/voice` (multipart)
   - 손글씨: `POST /api/tale/keyword/handwrite` (multipart)
2. 음성/손글씨는 Spring이 AI(FastAPI)에 중계하여 텍스트 추출.
3. 유저가 최종 단어 확정 시 `POST /api/tale/submit/keyword`.
4. `TaleService.keywordSubmit`이 Redis의 해당 TaleMember 키워드 갱신 후,
   4명 모두 완료되면 Spring이 AI로 동화 생성 요청(`/gen/tale`).

면접 포인트: “입력 인식(ASR/OCR)과 최종 확정은 분리되어 있어 UX 리트라이가 쉽다.”

## C. 동화 텍스트/프롬프트/음성 생성 → 그림 그리기 지시 배포

1. AI 동화 생성 응답 수신 시 `AIServerRequestService.handleGenerateTaleResponse` 실행.
2. 생성된 page 정보를 `TaleService.saveTaleText`로 Redis에 저장.
3. 누가 어떤 문장을 그릴지 `SentenceOwnerPair`를 `/topic/tale/{roomId}`로 브로드캐스트.
4. 동시에 백그라운드로
   - 확산 프롬프트 생성(`/gen/diffusion-prompts`) → Redis 저장
   - 각 페이지 TTS(`/gen/script-read`) → S3 업로드 URL Redis 저장
5. `verifyTaleMaking(roomId)`가 음성/원본그림/프롬프트 4개씩 준비됐는지 체크.

면접 포인트: “텍스트/프롬프트/TTS를 병렬화하고, 최종 단계 진입은 조건 충족 게이트로 제어.”

### `/topic/tale/{roomId}` 브로드캐스트 이후 클라이언트 분배 방식

서버는 `SentenceOwnerPair[]`(owner, order, sentence)를 **방 전체에 동일 payload로 브로드캐스트**합니다.  
이후 각 클라이언트가 프론트에서 자신의 모드/아이디 기준으로 다시 분배합니다.

- 1) 공통 수신
  - `playStore.setSubscribeTale(roomId)`에서 `/topic/tale/{roomId}`를 구독하고,
    수신된 배열을 `drawDirection` 상태로 저장합니다.
- 2) 정렬(공통)
  - `TaleSentenceDrawing`에서 `drawDirection`을 `order` 오름차순으로 정렬해
    페이지 순서를 고정합니다.
- 3) 멀티모드 분배
  - `owner === memberId` 조건으로 **내 문장 1개**를 선택(`multiModeSentences`).
  - 따라서 각 플레이어는 자신에게 할당된 한 문장만 보고 그립니다.
- 4) 싱글모드 분배
  - 문장 배열 전체를 순서대로 사용(`singleModeSentences[currentStep]`).
  - 한 사용자가 4개 문장을 차례대로 그리며 `addPage()`로 다음 문장으로 이동합니다.

즉, 백엔드는 “전원에게 같은 할당표”를 보내고, 프론트가 “내 역할만 추출(멀티) / 전체 순회(싱글)”로 최종 분배를 완성합니다.

## D. 손그림 제출 → AI 재탄생

1. 프론트가 `POST /api/tale/submit/picture` (multipart) 제출.
2. Spring `saveHandPicture`:
   - 원본 손그림을 S3 저장 후 Redis TaleMember `originImg` 갱신
3. 4명 제출 완료 시 Spring이 AI 그림변환 요청(`/gen/upgrade-handpicture`)을 각 페이지별 전송.
4. FastAPI는 외부 이미지 서버 호출 후 콜백을 받고,
   다시 Spring `/api/tale/submit/ai-picture`로 웹훅 전송.
5. Spring `saveAIPicture`가 최종 AI 그림 URL을 MySQL TaleMember.img에 저장.
6. 모든 조건 완료 시 `verifyTaleMaking`이
   - Redis 임시값을 MySQL로 flush
   - Redis 정리
   - `/topic/tale/{roomId}/finish` 브로드캐스트

면접 포인트: “AI 처리 완료를 polling이 아니라 webhook + 내부 검증 로직으로 종결 처리한다.”

---

## 4) 프론트 상태관리(Zustand) 기준 통신 책임 분리

- `roomStore`
  - WebSocket 연결/구독/방 입퇴장/시작 트리거 담당
  - 룸 단위 실시간 상태(currentRoom, participants, rawTale) 유지
- `playStore`
  - 제작 단계 도메인(키워드 입력/최종 제출/그림 제출) API 호출 담당
  - `/topic/tale/{roomId}` 구독으로 drawDirection 수신
  - `/topic/tale/{roomId}/finish`로 완료 전환

면접 포인트: “소켓 수명주기(룸)와 제작 액션(플로우)을 store 분리해 결합도를 낮췄다.”

---

## 5) Redis/MySQL 역할 분담 (왜 이렇게 했는가)

- **Redis(빠른 임시 상태)**
  - 방 인원 변동, 제작 중간 산출물(키워드/음성/프롬프트/원본그림)
  - 다중 사용자 동시편집에 적합
- **MySQL(최종 영속화)**
  - 완성 시점에 `saveTaleFromRedis`로 페이지 데이터 확정 저장
  - 조회, 히스토리, 갤러리 등 정합성 중심 기능 지원

면접 포인트: “실시간 협업은 Redis, 완료 스냅샷은 MySQL로 CQRS-lite 성격 분리.”

---

## 6) 기술 면접용 핵심 답변 템플릿

### Q1. 왜 소켓 + REST를 함께 썼나요?
- 실시간 동기화(방 입장/상태 전환)는 STOMP가 적합하고,
  파일 업로드/AI 호출은 HTTP multipart가 더 표준적이라 프로토콜을 분리했습니다.

### Q2. 4명이 동시에 편집할 때 일관성은 어떻게 보장하나요?
- 단계별로 Redis에 participant 단위 상태를 저장하고,
  `keyword 4명 완료`, `voice/origin/prompt 4개 완료` 같은 카운트 기반 게이트를 통과해야
  다음 단계로 넘어가게 했습니다.

### Q3. AI 완료 시점은 어떻게 감지하나요?
- FastAPI 콜백(webhook)으로 Spring `submit/ai-picture`를 호출하고,
  Spring에서 저장 후 `verifyTaleMaking`으로 전체 완료 여부를 확정합니다.

### Q4. 장애가 나면 어디가 병목인가요?
- 외부 AI/이미지 생성 지연이 크므로, 내부는 비동기 요청 + webhook 구조로 막힘을 줄였고,
  최종 완료 이벤트만 소켓으로 push합니다.

---

## 7) 코드 기준 근거 포인트(빠른 참조)

- STOMP 설정: `WebSocketConfig`
- 룸 소켓 엔드포인트: `RoomController`, `RoomService`
- 제작 REST API: `TaleController`
- 제작 오케스트레이션/검증: `TaleService`
- AI 연동 및 웹소켓 단계 알림: `AIServerRequestService`
- 프론트 소켓 룸 관리: `frontend/src/store/roomStore.js`
- 프론트 제작 플로우: `frontend/src/store/tale/playStore.js`
- 프론트 API 라우팅: `frontend/src/apis/tale/taleAxios.js`
- AI webhook 설정/전송: `AI/config/config.py`, `AI/app/core/picture.py`, `AI/app/routers/submit.py`
