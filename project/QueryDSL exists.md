# QueryDSL exists

## 목차

[문제 인식](#문제-인식)

[count VS exists](#count-vs-exists)

[QueryDSL exists](#querydsl-exists-1)

[QueryDSL exists 주의 사항](#querydsl-exists-주의-사항)

[QueryDSL count, exists 쿼리 실행 비교](#querydsl-count-exists-쿼리-실행-비교)

[결론](#결론)

## 문제 인식

채팅방의 메시지를 가져오기 위해서는 채팅방이 존재하는지, 해당 채팅방의 참여자인지를 확인해야 합니다.<br>
기존에는 이 작업을 count 쿼리를 통해서 조건에 맞는 데이터의 개수를 세고, 그 결과가 0보다 크다면 true를 반환하여 처리했습니다.

그러나, 현재 프로젝트에서 사용하는 채팅방은 1:1 채팅으로, 중복되지 않는 고유한 값을 가지고 있습니다.<br>
즉, 특정 채팅방에 대한 조건을 만족하는 데이터는 하나만 존재하며, 그 데이터만 찾으면 되지만, count 쿼리는 모든 데이터를 조회하기 때문에 성능상 문제가 발생할 수 있다고 판단하였습니다.

## count VS exists

count는 조건에 맞는 모든 데이터를 끝까지 확인한 후 개수를 반환합니다.<br>
즉, 조건을 만족하는 데이터가 첫 번째로 발견된다 하더라도 나머지 데이터를 계속 확인해야 하기 때문에 성능 저하를 유발할 수 있습니다.<br>
특히, 조건을 만족하는 데이터가 앞에 있을수록 이러한 성능 저하가 더 심하게 나타날 수 있습니다.

exists 쿼리는 조건에 맞는 첫 번째 데이터를 찾는 순간 쿼리가 종료됩니다.<br>
즉, 필요한 최소한의 데이터만 조회하고 더 이상 연산을 진행하지 않기 때문에 count 보다 성능이 개선될 수 있습니다.

## QueryDSL exists

이러한 성능 문제를 해결하기 위해 exists를 사용하려고 했으나 QueryDSL에서의 exists() 메서드는 실제로 SQL의 EXISTS가 아닌 COUNT를 사용하고 있습니다.<br>
즉, QueryDSL의 exists() 메서드는 기대하는 성능 최적화를 제공하지 못하는 상황이었습니다.

이를 해결하기 위해 exists 기능을 직접 구현해야 겠다고 생각했습니다.

exists가 count보다 성능이 좋은 이유는 조건에 해당하는 데이터를 1개만 찾으면 바로 쿼리를 종료하기 때문인 걸 생각하였고, limit 1 을 사용한 조회 쿼리를 통해 조건에 맞는 첫 번째 데이터만 가져오도록 설계하였습니다.<br>
이렇게 하면 조건에 맞는 첫 번째 데이터만 조회하게 되어 성능이 exists와 유사하게 개선될 수 있습니다.

## QueryDSL exists 주의 사항

기존의 count 방식에서는 결과를 0과 비교하여 존재 여부를 확인했지만, exists 방식에서는 결과가 없을 경우 0이 아니라 null이 반환되므로, null 체크를 통해 존재 여부를 판단해야 합니다.

## QueryDSL count, exists 쿼리 실행 비교

### QueryDSL count

- **QueryDSL count**

  ![querydslCount.PNG](../images/querydslCount.PNG)

- **하이버네이트 결과**

  ![querydslCountHibernate.PNG](../images/querydslCountHibernate.PNG)

### QueryDSL exists

- **QueryDSL exists**

  ![querydslExists.PNG](../images/querydslExists.PNG)

  ![querydslFetchFirst.PNG](../images/querydslFetchFirst.PNG)

  fetchFirst() 메서드를 확인하면 limit 1을 사용하는걸 확인할 수 있습니다.

- **하이버네이트 결과**

  ![querydslExistsHibernate.PNG](../images/querydslExistsHibernate.PNG)

## 결론

count와 exists는 데이터 조회에서 매우 큰 성능 차이를 유발할 수 있습니다.<br>
특히 데이터 개수가 많고 조건에 맞는 데이터가 앞에 있을 경우 exists는 첫 번째 데이터를 찾는 즉시 쿼리가 종료되므로 count보다 성능이 좋습니다.

QueryDSL에서는 직접 exists를 구현하여 이러한 성능 최적화를 적용할 수 있으며, limit 1을 활용한 조회 쿼리가 그 방법이 될 수 있습니다.