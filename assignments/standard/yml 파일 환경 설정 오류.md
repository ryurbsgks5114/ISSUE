# yml 파일 환경 설정 ISSUE

## 목차

[개요](#개요)

[ISSUE](#issue)

[해결 과정](#해결-과정)

[해결 방법](#해결-방법)

## 개요

스탠다드 실습반 첫 주차 과제의 요구 조건 중 하나인, 서버 환경 설정 파일인 application.yml을 local, dev로 나누고 datasource를 각각 적용해 보는 과정에서 발생한 ISSUE에 대해서 다룹니다.

## ISSUE

1. Caused by: java.sql.SQLSyntaxErrorException: Table 'member' already exists

2. java.sql.SQLException: Field 'married_count' doesn't have a default value

과제를 해결하기 위해 application-dev.yml, application-location.yml 파일을 추가하였고 application.yml 파일을 수정하였습니다.

기존 application.yml
```
spring:
  application:
    name: ttou-gayeon
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:ttougayoen;MODE=MYSQL;DATABASE_TO_LOWER=TRUE
    username: sa
    password:
    hikari:
      maximum-pool-size: 4

  jpa:
    database: h2
    hibernate:
      ddl-auto: create
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl

  h2:
    console:
      enabled: true
      path: /h2-console

  sql:
    init:
      mode: always
      schema-locations: classpath:schema.sql
```

수정 application.yml
```
spring:
  application:
    name: ttou-gayeon
  profiles:
    active: dev

  jpa:
    hibernate:
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl

  h2:
    console:
      enabled: true
      path: /h2-console

  sql:
    init:
      mode: always
      schema-locations: classpath:schema.sql
```

application-dev.yml
```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/standardassignment
    username: root
    password: rndus0324
    hikari:
      maximum-pool-size: 4

  jpa:
    database: mysql
    hibernate:
      ddl-auto: create
```

application-location.yml
```
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:ttougayoen;MODE=MYSQL;DATABASE_TO_LOWER=TRUE
    username: sa
    password:
    hikari:
      maximum-pool-size: 4

  jpa:
    database: h2
    hibernate:
      ddl-auto: create
```

파일을 수정 후에 첫 테스트에서는 문제없이 작동하였지만, 재실행 후에 테스트를 진행하려고 하니 1번 오류가 발생하였다.

member 테이블이 이미 존재한다는 오류여서 다른 부분은 잘 작동하는지 확인하기 위해 member 테이블을 삭제한 후에 재실행하였고, swagger-ui를 통해 member post API 요청을 하였는데 2번 오류가 발생하였다.

## 해결 과정

1번 오류를 해결하기 위하여 구글링을 해보니 application-dev.yml 파일의 ddl-auto: create을 update 또는 none으로 변경하라는 글이 있어 해보았지만, 똑같은 에러가 계속 발생하였다.

2번 오류는 married_count 필드에 값이 없다는 내용이어서 첫 번째로 swagger-ui에서 API 요청을 할 때, 값을 정상적으로 보내는지 확인하고, 디버깅을 통해 request에도 값이 정상적으로 들어오는지 확인하였다.<br>
값이 제대로 들어오는 걸 확인하여서 member Entity에 제대로 값이 매핑 되는지 디버깅을 통해서 확인하였는데 모든 값은 정상적으로 매핑 되었다.

도저히 혼자 해결이 안 되어서 튜터님께 찾아가 상황을 설명해 드리고 피드백을 받았다.

P.S. 튜터님께서 이런 오류를 일부러 만드셨고, 혼자 해결하면서 공부를 해보고, 질문을 오게끔 한 과제였는데, 아무도 찾아오지 않고 나만 찾아와서 좀 당황스러웠다고 하신다.

## 해결 방법

결론적으로 yml 파일의 설정으로 인한 오류였다.

application.yml
```
spring:
  application:
    name: ttou-gayeon
  profiles:
    active: dev

  h2:
    console:
      enabled: true
      path: /h2-console
```

application-location.yml
```
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:ttougayoen;MODE=MYSQL;DATABASE_TO_LOWER=TRUE
    username: sa
    password:
    hikari:
      maximum-pool-size: 4

  jpa:
    database: h2
    hibernate:
      ddl-auto: create

  sql:
    init:
      mode: always
      schema-locations: classpath:schema.sql
```

application.yml 파일에서 설정된 아래 코드 부분이 1번 오류를 발생하고 있었다.
```
sql:
    init:
      mode: always
      schema-locations: classpath:schema.sql
```

application.yml 파일에서 설정된 아래 코드 부분이 2번 오류를 발생하고 있었다.
```
jpa:
    hibernate:
      naming:
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```