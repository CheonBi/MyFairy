# 2025-01-13

## 기능

- **화상 접속 롤플레잉**: 사용자가 화상으로 만나 역할놀이 및 더빙을 함께 진행합니다.
- **영상 녹화**: 사용자가 진행하는 롤플레잉 영상은 녹화되어 저장되며, 후속 처리를 위해 사용될 수 있습니다.

## 기술 비교 표

| 기술                 | Latency (지연 시간)                           | Scalability (확장성)                               | Encoder/Player & Browser Support (브라우저 지원 및 플레이어)           | Cloud Support (클라우드 지원)                          | API Support (API 지원)                          |
|--------------------|---------------------------------------------|------------------------------------------------|----------------------------------------------------------|--------------------------------------------------|------------------------------------------------|
| **WebRTC**          | 매우 낮음 (실시간 통신에 최적화)               | 중간 ~ 높음 (P2P 방식, SFU/MCU 필요)              | 대부분의 최신 브라우저 지원 (Chrome, Firefox, Safari, Edge 등)     | 자체 호스팅 또는 클라우드 서비스 가능                    | WebRTC 표준 API 지원 (JavaScript, Android, iOS 등) |
| **OpenVidu**        | 낮음 ~ 중간 (WebRTC 사용)                    | 매우 높음 (SFU 기반, 서버 확장 용이)               | 주요 브라우저에서 WebRTC를 기반으로 지원                      | 클라우드 및 자체 서버에서 호스팅 가능                   | REST API 및 SDK 제공 (Java, Java, Python 등)    |
| **Kurento**         | 보통 (서버 기반 처리로 지연 증가 가능)         | 매우 높음 (Media Server 기반 확장성)             | H.264, VP8, VP9 지원 (서버 측 트랜스코딩)                       | 클라우드 배포 가능 (Media Server)                        | REST API 및 WebRTC 지원 (Java, JavaScript 등)   |
| **Jitsi**           | 낮음 (WebRTC 기반)                          | 보통 (대규모는 제한적)                            | H.264, VP8, VP9 지원 (서버 측 트랜스코딩)                       | 클라우드 배포 가능 (Jitsi 서버)                         | REST API 및 WebRTC 지원 (Java, Java 등)         |
| **Amazon Chime SDK**| 낮음 (WebRTC 기반)                          | 우수 (AWS 인프라 기반 확장성)                    | H.264, VP8 지원 (서버 측 트랜스코딩)                           | AWS 클라우드 기반 지원                                  | AWS SDK 제공, 다양한 API와 통합 가능            |
| **Daily.co**        | 낮음 (WebRTC 기반)                          | 우수 (클라우드 기반 확장성)                      | H.264, VP8 지원 (서버 측 트랜스코딩)                           | 클라우드 기반 (Daily.co 클라우드)                      | API 제공 (WebRTC 기반 회의, 녹화)               |
| **RTMP**            | 중간 ~ 높음 (실시간 방송에 적합, 지연 있음)    | 매우 높음 (대규모 방송에 적합)                   | 대부분의 RTMP 플레이어 지원 (예: Video.js, JWPlayer 등)         | 클라우드 기반 서비스 제공 가능 (AWS, Wowza 등)          | REST API 및 RTMP 서버 API 지원                  |
| **HLS**             | 중간 ~ 높음 (주로 스트리밍에 적합, 지연 있음)  | 매우 높음 (대규모 스트리밍에 최적화)               | HTML5 Video Player, 대부분의 모바일 브라우저 지원               | 클라우드 서비스 지원 (AWS Media Services 등)            | REST API 제공, 서버 측 처리 필요                 |
| **SIP**             | 중간 (VoIP/비디오 회의에서 적당)              | 중간 ~ 높음 (전화 시스템과 통합 가능)              | 제한적 지원 (특수 SIP 클라이언트 필요)                         | 클라우드 기반 VoIP 서비스 제공 (Twilio, Asterisk 등)   | SIP 서버 및 클라이언트 API 지원 (Java, C, Python 등) |
| **Dolby.io**        | 매우 낮음 (고품질 오디오 및 비디오)            | 중간 ~ 높음 (클라우드 기반)                       | 브라우저, iOS 및 Android에서 지원                              | Dolby.io 클라우드 서비스 사용                          | REST API 및 SDK 제공 (JavaScript, Python, C#, Java 등) |

## 기술 추천

### **OpenVidu**
- **장점**:
  - WebRTC 기반으로 높은 품질의 실시간 비디오/오디오 스트리밍 제공.
  - 녹화 기능을 제공하여 역할놀이 세션을 기록하고 저장 가능.
  - 비속어 필터링과 신고 시스템 등을 커스터마이즈 가능.
  - 서버 기반으로 스케일링이 용이하며, 고급 기능을 추가하기 좋음.
- **단점**:
  - 기본적인 기능 외에는 추가 개발이 필요함(예: 비속어 필터링, 신고 시스템 등).

### **Amazon Chime SDK**
- **장점**:
  - 클라우드 기반으로 고품질 비디오 및 오디오 제공.
  - 녹화 기능과 스크린샷/이미지 검열 기능이 쉽게 통합 가능.
  - 신고 시스템 및 사용자 관리 기능을 API를 통해 손쉽게 구현.
  - AWS의 강력한 확장성 및 안정성.
- **단점**:
  - Amazon AWS 환경에 익숙하지 않다면 설정 및 관리가 어려울 수 있음.

### **Daily.co**
- **장점**:
  - 클라우드 기반으로 빠르게 설정하고 확장할 수 있음.
  - WebRTC 기반으로 저지연 성능을 제공하며, API를 통한 통합이 용이.
  - 비디오 녹화 및 API를 통한 다양한 기능 확장이 가능.
- **단점**:
  - 커스터마이징이 필요한 경우 다소 제한적일 수 있음.

### **Jitsi**
- **장점**:
  - 오픈 소스로 제공되므로, 시스템에 대한 자유로운 커스터마이징이 가능.
  - WebRTC 기반으로 빠르고 안정적인 화상 회의 솔루션.
  - 녹화, 화면 공유 등 다양한 기능 제공.
- **단점**:
  - 클라우드 서비스를 직접 운영해야 하므로 관리 부담이 있을 수 있음.
  - 비속어 필터링, 신고 시스템은 추가 개발이 필요.

## 최종 추천

**OpenVidu**와 **Amazon Chime SDK**가 이 프로젝트에 가장 적합할 것으로 보입니다.

- **OpenVidu**는 저지연성과 확장성을 제공하면서, 녹화와 같은 기능을 손쉽게 커스터마이즈할 수 있습니다.
- **Amazon Chime SDK**는 AWS 인프라에서 제공하는 강력한 확장성과 고품질의 비디오 및 오디오를 제공하며, API를 통해 신고 시스템 및 부모 알림과 같은 기능을 구현할 수 있습니다. 클라우드 환경에서 통합이 용이한 장점도 있습니다.

따라서, **OpenVidu**를 기반으로 커스터마이징을 진행하거나, **Amazon Chime SDK**를 이용한 클라우드 기반 시스템을 고려해보는 것이 좋습니다.

# 2025-01-14

# Docker 설치 및 사용 가이드

## Docker Desktop 설치

터미널 명령어로 Docker를 조작할 수도 있지만, **Docker Desktop**이라는 공식 GUI 프로그램을 사용하면 Docker의 기능을 훨씬 더 편리하게 다룰 수 있습니다. Docker Desktop을 설치하면 Docker 엔진, Docker Compose, Docker CLI 등 필요한 모든 도구를 자동으로 설치해줍니다.

### 1. Docker Desktop 설치

1. 구글에서 **"Docker Desktop"**을 검색하여 공식 사이트에서 설치파일을 다운로드합니다.
2. 설치가 완료되면, Docker Desktop을 실행하여 Docker를 사용할 준비를 합니다.

### 2. 설치 시 주의사항

- **Windows**: 대부분의 경우 **AMD64**(x86-64) 버전을 선택하면 됩니다. 만약 ARM 기반 CPU를 사용하는 경우 ARM 버전을 선택합니다.
- **Mac**: **M1 이상**의 Mac을 사용한다면 **Apple Silicon** 버전을 선택합니다.
- **재시작**: Windows에서는 설치 후 컴퓨터를 재시작해야 할 수도 있습니다.

설치가 완료되면 Docker Desktop을 실행합니다.

---

## 자주 겪는 에러 및 해결 방법

### 1. "The network name cannot be found." 오류 (Windows)
- **해결 방법**:
  - **PowerShell**을 관리자 권한으로 실행한 후, 다음 명령어를 입력합니다:
    ```bash
    wsl --unregister docker-desktop
    ```
  - Docker Desktop을 종료하고 다시 실행해봅니다.

### 2. 시스템 리소스 부족 오류
- **해결 방법**:
  - 메모리(RAM)를 많이 사용하는 프로그램을 종료하거나, 컴퓨터를 재부팅한 후 Docker Desktop만 실행합니다.
  - 터미널에서 명령어로 Docker를 사용할 수 있습니다. Windows는 "PowerShell"에서, Mac은 "터미널"에서 Docker 명령어를 입력해 실행할 수 있습니다.

---

## Docker 이미지란?

### 이미지(Image)란?

Docker에서 **이미지**는 프로그램을 실행하기 위한 환경을 모두 포함한 파일입니다. 이미지는 **운영체제(OS)**, **필요한 라이브러리**, **소스 코드** 등을 포함하여, 실행 가능한 프로그램을 그대로 담고 있습니다. 이미지를 사용하면, 여러 환경에서 동일한 설정을 유지하면서 프로그램을 실행할 수 있습니다.

예를 들어, 웹 서버 프로그램을 개발한 후 이를 다른 서버에 배포하고 싶다면, 기존에는 다음과 같은 절차가 필요합니다:

1. AWS 등에서 서버를 하나 빌리고,
2. 코드를 서버로 전송하고,
3. 필요한 프로그램을 설치하고,
4. 웹 서버를 실행해야 합니다.

하지만 Docker를 사용하면:

1. 개발 환경과 코드를 모두 포함한 이미지를 만들고,
2. 이미지를 서버로 전송한 후,
3. 서버에서 이미지를 실행만 하면 됩니다.

Docker 이미지는 가벼운 실행 환경을 제공하므로 성능이나 효율성에 큰 영향을 주지 않습니다.

---

## Docker 이미지 실행하기

### 1. Docker Hub에서 이미지 다운로드

Docker Hub는 다양한 Docker 이미지가 저장된 중앙 저장소입니다. 여기서 원하는 이미지를 검색하고 다운로드할 수 있습니다.

1. **Docker Desktop**의 상단 검색창에서 Docker Hub에 올라온 공개 이미지를 검색할 수 있습니다.
2. 예시로 "hello-world" 이미지를 다운로드해 봅시다.
3. **Tag**는 이미지 버전과 유사한 개념입니다. 원하는 버전을 선택하고 **Pull** 버튼을 클릭하여 이미지를 다운로드합니다.

```bash
docker pull hello-world:latest
```

이 명령어를 실행하면, 이미지 안에서 실행되는 **컨테이너**가 생성됩니다. 컨테이너는 이미지를 실행한 **가상 컴퓨터**라고 할 수 있습니다.

다운로드한 **hello-world** 이미지는 간단한 "Hello World" 메시지를 출력하고 종료됩니다. 더 동적인 컨테이너를 실행하려면, **다이나믹한** 이미지를 실행해 봅시다.

---

## 더 복잡한 이미지 실행하기

### 1. 다이나믹한 웹 서버 이미지 실행

다음으로, `dockercloud/hello-world`라는 이미지를 실행해봅시다. 이 이미지는 **웹 서버**를 실행하는 이미지입니다.

1. **Docker Hub**에서 `dockercloud/hello-world` 이미지를 검색하고 다운로드합니다.
2. 다운로드한 이미지를 실행할 때, **포트 번호**를 지정해줍니다. 예를 들어, 아래 명령어를 사용하여 8080번 포트를 지정합니다.

    ```bash
    docker run -p 8080:80 dockercloud/hello-world
    ```

    이 명령어는 내 컴퓨터의 **8080번 포트**로 들어오면, 컨테이너 내부의 **80번 포트**로 요청을 전달하는 설정입니다.

3. 이제 웹 브라우저에서 `localhost:8080`에 접속하면 웹 페이지가 나타납니다. 이는 실행 중인 웹 서버가 해당 포트를 통해 요청을 처리하고 있기 때문입니다.

---

## 결론

- Docker 이미지는 **OS, 개발 환경, 소스 코드**를 하나의 파일로 묶어서 배포할 수 있는 방법입니다.
- 이미지를 실행하면 **컨테이너**가 생성되고, 컨테이너 안에서 해당 이미지가 실행됩니다.
- Docker는 개발 환경을 표준화하고, 배포 과정을 간소화하는 강력한 도구입니다.

# 2025-01-16

# Nginx에 대한 자세한 설명 (대학교 학부 졸업생 수준)

## 1. Nginx란 무엇인가?

Nginx는 **오픈소스** 웹 서버 소프트웨어로, 웹 서버뿐만 아니라 리버스 프록시 서버, 로드 밸런서, HTTP 캐시 등의 다양한 기능을 수행할 수 있습니다. 원래는 **고성능 웹 서버**를 목표로 설계되었고, 이후에는 다양한 용도로 확장되었습니다. Nginx는 특히 **비동기식 이벤트 기반 처리 방식**으로 높은 성능을 자랑합니다.

### 주요 특징
- **가벼운 리소스 사용**: 메모리와 CPU 자원을 적게 사용하면서도 매우 높은 처리 성능을 발휘합니다.
- **비동기식 이벤트 처리**: 한 번에 많은 클라이언트의 요청을 효율적으로 처리할 수 있습니다.
- **고가용성**: 서버가 다운되지 않도록 설정을 통해 장애 대응이 가능합니다.
- **로드 밸런싱 및 리버스 프록시**: 여러 서버에 트래픽을 분산시키거나, 요청을 다른 서버로 전달하는 기능이 있습니다.
- **빠른 정적 파일 처리**: 정적 파일 (HTML, 이미지, CSS 등)을 매우 빠르게 서빙합니다.

---

## 2. Nginx의 주요 기능

### 2.1. 웹 서버 기능
Nginx는 웹 서버로서 가장 기본적인 역할을 합니다. 웹 서버는 클라이언트(웹 브라우저)의 요청을 받아 웹 페이지를 제공하는 역할을 합니다. Nginx는 요청을 처리하고, 정적 파일(예: HTML, CSS, 이미지 등)을 클라이언트에게 반환합니다.

#### 특징
- **정적 파일 서빙**: Nginx는 정적 파일을 매우 빠르게 서빙할 수 있습니다.
- **디렉터리 리스팅**: 특정 디렉터리에 파일이 없을 경우, 디렉터리 내 파일 목록을 자동으로 생성하여 보여줄 수 있습니다.
- **압축 및 캐싱**: 클라이언트의 요청에 대해 데이터를 압축하거나 캐싱하여 응답 속도를 향상시킬 수 있습니다.

### 2.2. 리버스 프록시 서버
리버스 프록시 서버는 클라이언트의 요청을 받아 다른 서버로 전달하는 역할을 합니다. 이때 클라이언트는 실제로 리버스 프록시 서버와만 상호작용하고, 다른 서버의 세부 정보는 알 수 없습니다.

#### 특징
- **보안 향상**: 클라이언트는 실제 서버와 직접 연결되지 않기 때문에 보안이 강화됩니다.
- **로드 밸런싱**: 여러 서버로 요청을 분배하여 서버에 부하를 고르게 분산시킬 수 있습니다.
- **서버 숨기기**: 내부 서버의 정보나 구조를 외부에 숨길 수 있습니다.

### 2.3. 로드 밸런서
로드 밸런싱은 여러 서버에 트래픽을 분배하여 성능을 최적화하고, 서버에 과부하가 걸리지 않도록 합니다. Nginx는 로드 밸런서를 내장하고 있어 여러 서버 간에 트래픽을 분배하는 역할을 합니다.

#### 주요 로드 밸런싱 방식
- **라운드 로빈 (Round Robin)**: 요청을 순차적으로 각 서버에 분배합니다.
- **IP 해시 (IP Hash)**: 클라이언트의 IP 주소를 기준으로 요청을 특정 서버에 할당합니다.
- **최소 연결 (Least Connections)**: 가장 적은 수의 연결을 가진 서버에 요청을 전달합니다.

### 2.4. HTTP 캐시
Nginx는 웹 서버가 제공하는 콘텐츠를 캐시하여 성능을 개선할 수 있습니다. 캐시된 데이터는 빠르게 제공될 수 있기 때문에, 반복적인 요청이 많은 경우 매우 유용합니다.

#### 캐시 활용 예시
- **정적 파일 캐시**: 자주 변경되지 않는 정적 파일을 캐시하여 빠르게 전달합니다.
- **동적 콘텐츠 캐시**: 동적 웹 페이지의 결과를 일정 기간 동안 저장하여 서버 부하를 줄이고 응답 시간을 단축시킵니다.

### 2.5. SSL/TLS 종료
Nginx는 SSL/TLS를 지원하여 HTTPS 요청을 처리할 수 있습니다. 클라이언트와의 연결을 안전하게 암호화하고, 서버 간의 통신도 보호합니다. SSL 종료(SSL Termination)란 암호화된 HTTPS 트래픽을 Nginx가 복호화하고, 내부 서버는 HTTP로 통신하는 방식입니다.

---

## 3. Nginx 설정 파일 구조

Nginx의 설정 파일은 `/etc/nginx/nginx.conf`가 기본 파일이며, 이 파일 안에서 서버와 관련된 설정을 할 수 있습니다. 설정 파일은 **블록 구조**로 이루어져 있으며, 각 블록은 중괄호 `{}`로 감싸져 있습니다.

### 3.1. 주요 설정 파일 구조
1. **http 블록**: HTTP 서버와 관련된 설정을 포함합니다. 대부분의 설정은 이 블록 안에 작성됩니다.
2. **server 블록**: 각각의 가상 서버 설정을 포함합니다. 보통 하나의 `http` 블록 내에 여러 개의 `server` 블록이 있을 수 있습니다.
3. **location 블록**: 요청 URL을 기반으로 동작하는 세부 설정을 정의합니다. 예를 들어, 특정 URL에 대해서만 특정 동작을 하도록 설정할 수 있습니다.

#### 기본 설정 예시
```nginx
http {
    server {
        listen 80;  # HTTP 포트 80을 사용
        server_name example.com;  # 도메인 이름 설정

        location / {
            root /var/www/html;  # 웹 서버의 루트 디렉터리 설정
            index index.html;    # 기본 index 파일 설정
        }

        location /images/ {
            root /var/www/images;  # /images 경로에 대한 별도 설정
        }
    }
}
```

---

## 4. Nginx 설치 및 사용 방법

### 4.1. 설치 (Ubuntu 기준)
Nginx를 설치하는 방법은 운영 체제에 따라 다를 수 있지만, 일반적으로 리눅스에서는 다음 명령어로 설치할 수 있습니다.

```bash
sudo apt update
sudo apt install nginx
```

### 4.2. Nginx 실행 및 상태 확인
설치 후 Nginx 서버를 시작하거나 중지할 수 있습니다.

- **Nginx 시작**: 
  ```bash
  sudo systemctl start nginx
  ```
 
- **Nginx 중지**:
  ```bash
  sudo systemctl stop nginx
  ```

- - **Nginx 상태 확인**:

  ```bash
  sudo systemctl status nginx
  ```

### 4.3 Nginx 설정 파일 테스트
설정 파일을 수정한 후, 반드시 설정 파일에 오류가 없는지 확인해야 합니다. 이를 위해 다음 명령어를 사용할 수 있습니다.
```bash
sudo nginx -t
```

### 4.4 설정  적용
설정 파일에 오류가 없다면, Nginx를 다시 로드하여 설정을 적용합니다.
```bash
sudo systemctl reload nginx
```

---

### 5. Nginx vs Apache
Nginx와 Apache는 둘 다 매우 인기 있는 웹 서버이지만, 몇 가지 차이점이 있습니다.

| 특징           | Nginx                         | Apache                        |
|----------------|-------------------------------|-------------------------------|
| **성능**       | 비동기식으로 높은 처리 성능    | 동기식으로 상대적으로 낮은 성능 |
| **리소스 사용**| 적은 리소스 사용              | 많은 리소스 사용               |
| **구성**       | 설정 파일이 간단하고 직관적   | 설정 파일이 복잡할 수 있음    |
| **기능**       | 리버스 프록시, 로드 밸런싱 등 | 동적 콘텐츠 처리에 강함       |

- **Nginx**는 주로 정적 파일 서빙과 리버스 프록시 서버, 로드 밸런서로 사용됩니다.
- **Apache**는 동적 콘텐츠 처리에 강하며, 모듈을 사용하여 다양한 기능을 추가할 수 있습니다.

---

## 6. 결론

Nginx는 그 성능과 효율성 덕분에 대규모 웹 애플리케이션 및 서비스에서 널리 사용됩니다. 비동기식 이벤트 기반 처리 방식으로 높은 처리량을 자랑하며, 다양한 기능을 제공하여 웹 서버, 리버스 프록시, 로드 밸런서 등 여러 역할을 수행할 수 있습니다. Nginx는 설정이 간단하고 확장성이 높아 많은 환경에서 유용하게 활용됩니다.

# 2025-01-16

# 도커 시작

### Spring Boot : application.properties

```yaml
spring.application.name=proj
server.port=8080

# DataSource (MySQL)
spring.datasource.url=jdbc:mysql://mysql-container:3306/ssafy
spring.datasource.username=ssafy
spring.datasource.password=ssafy
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA 
#spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true

spring.jpa.open-in-view=false
```

- `hibernate.dialect` 관련 설정 변경:
    
    현재 설정에서 `spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect`가 사용되고 있습니다. 이 설정은 Hibernate 6.x부터 더 이상 필요하지 않으며, 대신 **자동으로 MySQL Dialect가 선택됩니다**. 게다가, `MySQL8Dialect`는 deprecated된 상태입니다. 그래서 다음과 같이 수정해야 합니다:
    
- **`spring.jpa.database-platform` 설정 제거**: 이 설정을 제거하면 Hibernate가 자동으로 올바른 Dialect를 선택합니다.
    
    ```
    # spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
    ```
    

### 2. `spring.jpa.open-in-view` 설정 추가:

`spring.jpa.open-in-view`가 기본적으로 `true`로 설정되어 있기 때문에 데이터베이스 쿼리가 뷰 렌더링 시점에 실행될 수 있다는 경고가 발생합니다. 이를 방지하려면 `false`로 설정하는 것이 좋습니다:

- **`spring.jpa.open-in-view=false` 추가**:
    
    ```
    spring.jpa.open-in-view=false
    ```
    

## 선택 사항

### 방법1-1. jar 빌드하기

```yaml
gradlew build
```

### 1-2. Dockerfile 작성

```docker
FROM openjdk:17-jdk

WORKDIR /app

COPY build/libs/*SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "app.jar"]

# docker build -t proj:v1.0
# -t : 태그를 지정
# proj : 이미지 이름
# v1.0 태그 이름
```

### 방법 2-1. Dockerfile에서 jar 파일 만들어서 바로 진행

**Multi-stage Build**

- **효율적인 이미지 크기**: 빌드 도구(`gradle`, `git`, `maven` 등)를 포함한 이미지가 최종 이미지에 포함되지 않아서, 결과적으로 더 작은 런타임 이미지를 만들 수 있습니다.
- **보안성 강화**: 빌드 환경을 런타임 환경에서 분리함으로써, 개발 환경에만 필요한 도구를 포함시키지 않게 됩니다.
- **빌드 과정의 분리**: 빌드와 실행을 명확하게 분리함으로써, 나중에 빌드 단계를 수정하거나 재사용하는데 용이합니다.

### Spring Boot 컨테이너 만들기

```docker
docker run --name server-container -d -p 8080:8080 springserver:1
```

### Nginx 설정 파일 작성

```yaml
server {
        listen 80;
        location / {
            proxy_pass http://server-container:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
}
```

### Nginx Dockerfile 작성

```docker
FROM nginx:1.27-alpine

COPY backconfig.conf /etc/nginx/conf.d/backconfig.conf
RUN rm /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### MySQL 이미지 설치

```docker
docker pull mysql
```

### Docker MySQL 컨테이너 실행

```docker
docker run --name <컨테이너명> -e MYSQL_ROOT_PASSWORD=<password> -d -p 3306:3306 mysql:latest
```

**● --name <container_name>** : <container_name> 이름의 컨테이너를 실행한다.

● **-e** : 컨테이너 내에서 사용할 환경변수를 설정

● **-e MYSQL_ROOT_PASSWORD=<password>** : MySQL의 root 권한의 비밀번호를 <password>로 설정한다.

● **-d** : detach 모드로 컨테이너가 실행된다. 컨테이너가 백그라운드로 실행된다고 보면 된다.

● **-p** <호스트 포트> <컨테이너 포트> : 호스트와 컨테이너의 포트를 연결한다 **(Host에 DB가 이미 실행되고 있다면 3306 포트는 사용중이므로 변경)**

● **mysql:latest** : 컨테이너에 사용할 이미지

```docker
docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=0000 -d -p 3307:3306 mysql:latest
```

- MySQL 이 3307 포트 사용중이라 3307로 변경

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/6cd66313-3583-441e-a126-a690f70eccfb/47ed9827-24e7-4778-9644-314e88f8acb7/image.png)

- Exec로 가서 MySQL 명령문 실행

```docker
# IDE 터미널일 경우
docker exec -it mysql-container mysql -u root -p

# Docker exec에선
mysql -u root -p

# 사용자 생성
CREATE USER 'ssafy'@'%' IDENTIFIED BY 'ssafy';

# 권한 부여
GRANT ALL PRIVILEGES ON *.* TO 'ssafy'@'%' WITH GRANT OPTION;

# 권한 적용
FLUSH PRIVILEGES;
```

### Docker-Commpose 파일

```docker
services:
  mysql-container:
    image: mysql:latest
    environment:
      MYSQL_ROOT_PASSWORD: 0000       # 루트 비밀번호 설정
      MYSQL_DATABASE: ssafy                   # 기본 데이터베이스 설정
      MYSQL_USER: ssafy                       # 사용자 설정
      MYSQL_PASSWORD: ssafy                   # 사용자의 비밀번호 설정
    ports:
      - 3307:3306
    networks:
      - mynet2
    restart: unless-stopped                   # MySQL 서버가 다운되면 자동으로 재시작
    volumes:
      - mysql-data:/var/lib/mysql             # 데이터 유지용 볼륨 설정

  server-container:
    image: springserver:1
    ports:
      - 8080:8080
#    environment: 환경 변수 설정
#      - name=value
    networks:
      - mynet1
      - mynet2
    restart: unless-stopped

  nginx:
    image: nginx:1
    ports:
      - 80:80
    restart: unless-stopped
    networks:
      - mynet1

networks:
  mynet1:
    driver: bridge
  mynet2:
    driver: bridge

volumes:
  mysql-data:
    driver: local
```
