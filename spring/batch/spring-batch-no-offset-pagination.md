# Spring Batch No-Offset Pagination 을 이용한 대용량 데이터 처리

## Overview
DB 에서 대량의 데이터를 조회할 때, 일반적으로 offset pagination 을 이용하곤한다. 그러나 offset 을 기반으로 조회하는 방식은 RDB 의 Index 구조상 offset 의 숫자가 커지면 성능이 급격히 저하될 수 밖에 없다. 때문에 꼭 offset 을 지정해서 조회하는 방식의 서비스가 아니라면, offset 을 생략한 pagination 을 도입하여 일관된 조회 성능을 보장해볼 수 있다. 흔히 no-offset pagination, cursor pagination 이라 불리는 그것이다.

## off





Spring Batch 를 이용해서 데이터 조회할 때, 일반적으로 오프셋 기반 페이지네이션(offset-based pagination)을 사용한다.
그러나 이 방법은 데이터베이스에서 오프셋이 커질수록 성능이 저하되는 문제가 있다. 

이를 해결하기 위해 No-Offset Pagination 기법을 사용할 수 있다.
