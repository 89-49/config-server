## Config Server 사용 안내

### 1. Config Server 주소

현재 Config Server는 GCP VM에 배포되어 있습니다.

```text
http://34.50.50.170:13100
```

Config Server는 각 서비스의 설정 파일을 중앙에서 관리하기 위한 서버입니다.

각 서비스는 자신의 로컬 `application.yml`에 모든 설정을 직접 작성하지 않고, Config Server를 통해 GitHub `configs` 저장소에 있는 설정을 받아옵니다.

---

## 2. Config Server 상태 확인

Config Server가 정상적으로 실행 중인지 확인하려면 아래 주소로 접근합니다.

```text
http://34.50.50.170:13100/actuator/health
```

정상이라면 아래와 비슷한 응답이 반환됩니다.

```json
{
  "status": "UP"
}
```

---

## 3. 서비스별 설정 조회 방법

Config Server 설정은 아래 형식으로 조회할 수 있습니다.

```text
http://34.50.50.170:13100/{서비스명}/{프로필}
```

예를 들어 `gateway-server`의 기본 설정을 확인하려면 아래 주소로 접근합니다.

```text
http://34.50.50.170:13100/gateway-server/default
```

`dev` 프로필 설정을 확인하려면 아래 주소로 접근합니다.

```text
http://34.50.50.170:13100/gateway-server/dev
```

다른 서비스도 같은 방식으로 조회할 수 있습니다.

```text
http://34.50.50.170:13100/user-service/default
```

```text
http://34.50.50.170:13100/product-service/default
```

```text
http://34.50.50.170:13100/trade-service/default
```

```text
http://34.50.50.170:13100/chat-service/default
```

---

## 4. 각 서비스에서 Config Server 연결 방법

각 서비스의 로컬 `application.yml`에는 Config Server 연결을 위한 최소 설정만 작성합니다.

```yml
spring:
  application:
    name: 본인-서비스명

  config:
    import: optional:configserver:http://34.50.50.170:13100
```

여기서 가장 중요한 값은 `spring.application.name`입니다.

이 값은 GitHub `configs` 저장소에서 어떤 서비스 설정 폴더를 읽을지 결정하는 이름입니다.

예를 들어 아래처럼 설정되어 있다면:

```yml
spring:
  application:
    name: user-service
```

Config Server는 GitHub `configs` 저장소에서 아래 경로를 찾습니다.

```text
user-service/application.yml
```

즉, `spring.application.name`과 GitHub `configs` 저장소의 서비스 폴더명이 반드시 일치해야 합니다.

---

## 5. 프로필을 사용하는 경우

`dev` 프로필을 사용하려면 각 서비스의 로컬 `application.yml`에 아래처럼 설정합니다.

```yml
spring:
  application:
    name: 본인-서비스명

  profiles:
    active: dev

  config:
    import: optional:configserver:http://34.50.50.170:13100
```

예를 들어 서비스명이 `user-service`이고 `dev` 프로필로 실행하면 Config Server는 아래 주소로 설정을 조회하는 것과 같은 방식으로 동작합니다.

```text
http://34.50.50.170:13100/user-service/dev
```

---

## 6. GitHub configs 저장소 수정 방식

서비스별 설정은 각 서비스 레포지토리의 로컬 `application.yml`에 모두 작성하지 않고, GitHub의 `configs` 저장소에서 관리합니다.

설정 저장소 주소는 아래와 같습니다.

```text
https://github.com/89-49/configs
```

각 팀원은 본인 서비스의 설정을 수정해야 할 때, `configs` 저장소에서 자기 서비스에 해당하는 `application.yml` 파일을 수정하면 됩니다.

예를 들어 `user-service` 담당자는 아래 파일을 수정합니다.

```text
user-service/application.yml
```

공통으로 사용하는 설정은 아래 파일에 작성합니다.

```text
common/application.yml
```

개발 환경 공통 설정은 아래 파일에 작성합니다.

```text
common/application-dev.yml
```

---

## 7. 현재 configs 저장소 구조

현재 Config Server가 바라보는 GitHub 설정 저장소 구조는 아래와 같습니다.

```text
configs
├── common
│   ├── application.yml
│   ├── application-dev.yml
│   ├── application-kafka.yml
│   ├── application-test.yml
│   └── application-topics.yml
│
├── gateway-server
│   └── application.yml
│
├── user-service
│   └── application.yml
│
├── product-service
│   └── application.yml
│
├── trade-service
│   └── application.yml
│
└── chat-service
    └── application.yml
```

`common` 폴더에는 여러 서비스가 공통으로 사용하는 설정을 작성합니다.

각 서비스별 폴더에는 해당 서비스에서만 사용하는 설정을 작성합니다.

---

## 8. Config Server의 설정 탐색 방식

현재 Config Server는 설정 저장소에서 아래 경로를 탐색합니다.

```yml
search-paths:
  - common
  - "{application}"
```

따라서 `user-service`가 설정을 요청하면 Config Server는 아래 경로를 확인합니다.

```text
common
user-service
```

`gateway-server`가 설정을 요청하면 아래 경로를 확인합니다.

```text
common
gateway-server
```

즉, 각 서비스의 `spring.application.name`과 `configs` 저장소의 서비스 폴더명이 일치해야 설정을 정상적으로 가져올 수 있습니다.

---

## 9. 설정이 정상 조회되는지 확인하는 방법

각 서비스를 실행하기 전에 Config Server에서 설정이 잘 조회되는지 먼저 확인할 수 있습니다.

예를 들어 `user-service` 설정은 아래 주소로 확인합니다.

```text
http://34.50.50.170:13100/user-service/default
```

`dev` 프로필 설정은 아래 주소로 확인합니다.

```text
http://34.50.50.170:13100/user-service/dev
```

정상이라면 응답의 `propertySources`가 비어 있지 않아야 합니다.

정상 예시:

```json
{
  "name": "user-service",
  "profiles": [
    "default"
  ],
  "propertySources": [
    {
      "name": "..."
    }
  ]
}
```

비정상 예시:

```json
{
  "name": "user-service",
  "profiles": [
    "default"
  ],
  "propertySources": []
}
```

`propertySources`가 비어 있으면 Config Server가 해당 서비스 설정 파일을 찾지 못한 것입니다.

이 경우 아래 내용을 확인해야 합니다.

```text
1. 각 서비스의 spring.application.name이 configs 저장소의 폴더명과 같은지 확인
2. configs 저장소에 해당 서비스 폴더가 있는지 확인
3. 해당 서비스 폴더 안에 application.yml이 있는지 확인
4. 프로필을 사용하는 경우 application-dev.yml 같은 파일명이 올바른지 확인
```

---

## 10. 설정 변경 반영 방식

설정을 변경해야 할 때는 GitHub `configs` 저장소에서 해당 서비스의 `application.yml` 파일을 수정한 뒤 push합니다.

기본 흐름은 아래와 같습니다.

```text
configs 저장소에서 본인 서비스 application.yml 수정
→ GitHub에 push
→ 해당 서비스 재시작
→ 변경된 설정 적용
```

예를 들어 `user-service` 설정을 변경하는 경우:

```text
configs/user-service/application.yml 수정
→ GitHub에 push
→ user-service 재시작
→ 변경된 설정 적용
```

현재는 각 서비스가 이미 실행 중인 상태에서 설정이 자동으로 즉시 반영되지는 않습니다.

가장 단순하고 확실한 방법은 설정 변경 후 해당 서비스를 재시작하는 것입니다.

추후 Actuator refresh, Spring Cloud Bus 등을 적용하면 서비스 재시작 없이 설정을 갱신하는 구조로 확장할 수 있습니다.

---

## 11. 주의사항

Config Server 주소는 아래와 같습니다.

```text
http://34.50.50.170:13100
```

각 서비스에서 Config Server를 사용할 때는 로컬 `application.yml`에 아래 설정을 추가해야 합니다.

```yml
spring:
  config:
    import: optional:configserver:http://34.50.50.170:13100
```

초기 개발 단계에서는 `optional:`을 붙여두는 것을 권장합니다.

```yml
spring:
  config:
    import: optional:configserver:http://34.50.50.170:13100
```

`optional:`을 붙이면 Config Server에 일시적으로 접근하지 못해도 애플리케이션이 바로 실패하지 않을 수 있습니다.

다만 Config Server 연결 실패를 빠르게 감지해야 하는 환경에서는 `optional:`을 제거할 수 있습니다.

```yml
spring:
  config:
    import: configserver:http://34.50.50.170:13100
```

---

## 12. 팀원 사용 요약

각 팀원은 본인 서비스의 로컬 `application.yml`에 아래 설정을 추가합니다.

```yml
spring:
  application:
    name: 본인-서비스명

  config:
    import: optional:configserver:http://34.50.50.170:13100
```

그다음 GitHub `configs` 저장소에서 본인 서비스 폴더의 `application.yml`을 수정하면 됩니다.

예시는 아래와 같습니다.

```text
user-service 담당자
→ configs/user-service/application.yml 수정

product-service 담당자
→ configs/product-service/application.yml 수정

trade-service 담당자
→ configs/trade-service/application.yml 수정

chat-service 담당자
→ configs/chat-service/application.yml 수정

gateway-server 담당자
→ configs/gateway-server/application.yml 수정
```

설정이 정상적으로 조회되는지는 아래 형식으로 확인합니다.

```text
http://34.50.50.170:13100/{서비스명}/{프로필}
```

예시:

```text
http://34.50.50.170:13100/user-service/default
```

```text
http://34.50.50.170:13100/gateway-server/default
```

응답의 `propertySources`가 비어 있지 않으면 Config Server 연동이 정상적으로 된 것입니다.
