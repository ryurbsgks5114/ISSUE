# SQL 쿼리 호출 최적화 및 성능 향상 테스트

## 목차

[문제 정의](#문제-정의)

[테스트 환경](#테스트-환경)

[테스트 방식](#테스트-방식)

## 문제 정의

현재 프로젝트에서는 Spring Data JPA의 JpaRepository 인터페이스를 사용하여 데이터를 조회하고 있습니다.

하지만, 복잡한 연관관계 데이터를 조회할 때는 JpaRepository 만으로는 한계가 있어, Stream map 메서드와 JpaRepository를 조합하여 데이터를 조회하고 있었습니다.

이러한 방식은 다중 쿼리 호출을 유발하여 성능 저하 문제가 발생할 수 있습니다.

이 문제를 해결하기 위해 QueryDSL을 적용하여, 서브쿼리와 LEFT JOIN을 사용해 다중 쿼리 호출을 하나의 쿼리로 통합하여 성능을 개선하려고 합니다.

## 테스트 환경

- **Java**
  - OpenJDK 17
- **Spring Framework**
  - Spring Boot 3.2.8
  - Spring Data JPA 3.2.8
  - QueryDSL JPA 5.0.0
- **Database**
  - PostgreSQL 14.12
- **Data**
  - User Data 100개
  - Post Data 100개
  - Product Data 100개
  - Review Data 100개
  - Wish Data 100개
- **OS**
  - Windows 10
- **CPU**
  - Intel(R) Core(TM) i7-7700 CPU @ 3.60GHz
- **Memory**
  - 16.0GB

## 테스트 방식

SQL 쿼리 실행 전후로 Spring AOP를 적용하여 메서드의 실행 시간을 측정합니다.

측정된 SQL 쿼리 실행 시간을 DB에 메서드가 호출된 위치와 실행 시간을 저장하여 관리합니다.

QueryDSL 적용 전후의 SQL 실행 시간을 비교하여 성능 향상 여부를 평가합니다.