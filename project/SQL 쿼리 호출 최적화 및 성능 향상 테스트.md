# SQL 쿼리 호출 최적화 및 성능 향상 테스트

## 목차

[문제 정의](#문제-정의)

[테스트 환경](#테스트-환경)

[테스트 방식](#테스트-방식)

[QueryDSL 적용 전](#querydsl-적용-전)

[QueryDSL 적용 후](#querydsl-적용-후)

[비교 분석](#비교-분석)

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

SQL 쿼리 실행 전후로 Spring AOP를 적용하여 메서드의 실행 시간을 10번 측정합니다.

측정된 SQL 쿼리 실행 시간을 DB에 메서드가 호출된 위치와 실행 시간을 저장하여 관리합니다.

QueryDSL 적용 전후의 SQL 실행 시간을 비교하여 성능 향상 여부를 평가합니다.

## QueryDSL 적용 전

<details>
<summary>내가 등록한 게시물 목록 조회</summary>

- **메서드 위치**: post.service.PostService.getMyPostList()
- **성능 분석**

![post.service.PostService.getMyPostList()-nosql](../images/post.service.PostService.getMyPostList()-nosql.PNG)

- **최소**: 28ms
- **최대**: 54ms
- **평균**: 40.3ms

</details>

<details>
<summary>내가 등록한 상품 목록 조회</summary>

- **메서드 위치**: product.service.ProductService.getMyProductList()
- **성능 분석**

![product.service.ProductService.getMyProductList()-nosql](../images/product.service.ProductService.getMyProductList()-nosql.PNG)

- **최소**: 15ms
- **최대**: 29ms
- **평균**: 20.2ms

</details>

<details>
<summary>내 상점 후기 목록 조회</summary>

- **메서드 위치**: review.service.ReviewService.getMyReview()
- **성능 분석**

![review.service.ReviewService.getMyReview()-nosql](../images/review.service.ReviewService.getMyReview()-nosql.PNG)

- **최소**: 9ms
- **최대**: 14ms
- **평균**: 11.4ms

</details>

<details>
<summary>내가 찜한 상품 목록 조회</summary>

- **메서드 위치**: wish.service.WishService.getMyWishList()
- **성능 분석**

![wish.service.WishService.getMyWishList()-nosql](../images/wish.service.WishService.getMyWishList()-nosql.PNG)

- **최소**: 13ms
- **최대**: 20ms
- **평균**: 16.3ms

</details>

<details>
<summary>채팅방 목록 조회</summary>

- **메서드 위치**: chat.service.ChatService.getChatRoom()
- **성능 분석**

![chat.service.ChatService.getChatRoom()-nosql](../images/chat.service.ChatService.getChatRoom()-nosql.PNG)

- **최소**: 28ms
- **최대**: 53ms
- **평균**: 38.7ms

</details>

## QueryDSL 적용 후

<details>
<summary>내가 등록한 게시물 목록 조회</summary>

- **메서드 위치**: post.service.PostService.getMyPostList()
- **성능 분석**

![post.service.PostService.getMyPostList()](../images/post.service.PostService.getMyPostList().PNG)

- **최소**: 9ms
- **최대**: 16ms
- **평균**: 12.5ms

</details>

<details>
<summary>내가 등록한 상품 목록 조회</summary>

- **메서드 위치**: product.service.ProductService.getMyProductList()
- **성능 분석**

![product.service.ProductService.getMyProductList()](../images/product.service.ProductService.getMyProductList().PNG)

- **최소**: 5ms
- **최대**: 8ms
- **평균**: 6.5ms

</details>

<details>
<summary>내 상점 후기 목록 조회</summary>

- **메서드 위치**: review.service.ReviewService.getMyReview()
- **성능 분석**

![review.service.ReviewService.getMyReview()](../images/review.service.ReviewService.getMyReview().PNG)

- **최소**: 3ms
- **최대**: 5ms
- **평균**: 4.1ms

</details>

<details>
<summary>내가 찜한 상품 목록 조회</summary>

- **메서드 위치**: wish.service.WishService.getMyWishList()
- **성능 분석**

![wish.service.WishService.getMyWishList()](../images/wish.service.WishService.getMyWishList().PNG)

- **최소**: 5ms
- **최대**: 8ms
- **평균**: 6.2ms

</details>

<details>
<summary>채팅방 목록 조회</summary>

- **메서드 위치**: chat.service.ChatRoomService.getChatRoom()
- **성능 분석**

![chat.service.ChatRoomService.getChatRoom()](../images/chat.service.ChatRoomService.getChatRoom().PNG)

- **최소**: 2ms
- **최대**: 4ms
- **평균**: 3.3ms

</details>

## 비교 분석

**성능 개선율** = ((기존 실행 시간 - 개선된 실행 시간) / 기존 실행 시간) * 100

<details>
<summary>내가 등록한 게시물 목록 조회</summary>

**QueryDSL 적용 후 약 68.98%의 성능 개선이 이루어졌습니다.**

|  | QueryDSL 적용 전 | QueryDSL 적용 후 |
| --- | --- | --- |
| 최소 실행 시간 | 28ms | 9ms |
| 최대 실행 시간 | 54ms | 16ms |
| 평균 실행 시간 | 40.3ms | 12.5ms |

</details>

<details>
<summary>내가 등록한 상품 목록 조회</summary>

**QueryDSL 적용 후 약 67.82%의 성능 개선이 이루어졌습니다.**

|  | QueryDSL 적용 전 | QueryDSL 적용 후 |
| --- | --- | --- |
| 최소 실행 시간 | 15ms | 5ms |
| 최대 실행 시간 | 29ms | 8ms |
| 평균 실행 시간 | 20.2ms | 6.5ms |

</details>

<details>
<summary>내 상점 후기 목록 조회</summary>

**QueryDSL 적용 후 약 64.04%의 성능 개선이 이루어졌습니다.**

|  | QueryDSL 적용 전 | QueryDSL 적용 후 |
| --- | --- | --- |
| 최소 실행 시간 | 9ms | 3ms |
| 최대 실행 시간 | 14ms | 5ms |
| 평균 실행 시간 | 11.4ms | 4.1ms |

</details>

<details>
<summary>내가 찜한 상품 목록 조회</summary>

**QueryDSL 적용 후 약 61.96%의 성능 개선이 이루어졌습니다.**

|  | QueryDSL 적용 전 | QueryDSL 적용 후 |
| --- | --- | --- |
| 최소 실행 시간 | 13ms | 5ms |
| 최대 실행 시간 | 20ms | 8ms |
| 평균 실행 시간 | 16.3ms | 6.2ms |

</details>

<details>
<summary>채팅방 목록 조회</summary>

**QueryDSL 적용 후 약 91.47%의 성능 개선이 이루어졌습니다.** 
(채팅방 목록 조회의 성능 개선율이 유독 높은 이유는 페이지네이션 적용 여부입니다. 추후 채팅방 목록 조회에도 페이지네이션을 적용할 예정입니다.)

|  | QueryDSL 적용 전 | QueryDSL 적용 후 |
| --- | --- | --- |
| 최소 실행 시간 | 28ms | 2ms |
| 최대 실행 시간 | 53ms | 4ms |
| 평균 실행 시간 | 38.7ms | 3.3ms |

</details>