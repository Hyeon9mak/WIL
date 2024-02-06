> 준우님이 정리해주신 내용 베이스. 다시 한번 재해석 할 것.
### **인덱스란?**

인덱스는 쿼리의 효율적인 실행을 지원합니다.
인덱스가 없을 경우에 쿼리 결과를 반환하기 위해 컬렉션의 모든 문서를 스캔해야 합니다.
하지만 적합한 인덱스가 존재하면 스캔하는 문서의 수를 줄일 수 있습니다.
인덱스는 읽기 작업의 성능을 향상시키지만, 쓰기 작업에는 성능에 부정적인 영향을 미칩니다.
즉, 읽기와 쓰기의 비율을 잘 고려해서 적용해야 합니다.
MongoDB의 인덱스는 내부적으로는 B-Tree 데이터 구조를 활용합니다.

### **기본적으로 지원하는 인덱스**

_id 필드에 고유 인덱스를 생성하며 해당 인덱스는 삭제할 수 없습니다.

### **인덱스 만드는 방법**

```bash
db.collection.createIndex( { name: -1 } )
```

내림차순으로 name이라는 필드에 인덱스를 생성하는 방법입니다.

```bash
db.<collection>.createIndex(
   { <field>: <value> },
   { name: "<indexName>" }
)
```

인덱스를 생성할때는 이름을 지정할 수도 있고 지정하지 않으면 기본적인 전략에 의해 만들어집니다.

기본 전략은 key, value사이에 언더스코어(_)가 붙어 name_-1 이라는 인덱스의 이름으로 생성될 것입니다.

### **인덱스 확인**

```bash
db.collection.getIndexes()
```

인덱스가 정상적으로 만들어졌는데 확인할 수 있습니다.

```bash
[
  { v: 2, key: { _id: 1 }, name: '_id_' },
  { v: 2, key: { name: -1 }, name: 'name_-1' }
]
```

기본적으로 id에 대한 인덱스가 존재하였고 name 필드에도 인덱스를 생성하기 때문에 존재하는 것을 확인할 수 있습니다.

### **인덱스 삭제**

```bash
db.<collection>.dropIndex("<indexName>")
```

### **인덱스의 타입**

**단일 필드에 적용하는 Single Field Index**

```bash
db.students.createIndex( { gpa: 1 } )
```

**여러개의 필드에 적용하는 Compound Index**

```bash
db.leaderboard.createIndex( { score: -1, username: 1 } )
```

복합인덱스에서는 정렬 순서에 따라 쿼리가 인덱스를 타지 않을 수 있어서 주의해야 합니다.

```bash
db.leaderboard.createIndex( { score: -1, username: 1 } ) // (1) 지원됨
db.leaderboard.find().sort( { score: 1, username: -1 } ) // (2) 지원됨
db.leaderboard.find().sort( { score: 1, username: 1 } ) // (3) 지원되지 않음
db.leaderboard.find().sort( { username: 1, score: -1, } ) // (4) 지원되지 않음 필드 순서 유의
```

(4) 번은 (1) 번과 무슨 차이야?라고 생각되지만 인덱스를 생성했던 필드의 순서를 유의해야 합니다.

**배열값이 포함된 필드에 적용하는 Multikey Index**

![https://blog.kakaocdn.net/dn/bU7Dj6/btsD4sciZNQ/Jiyj1FM0SZThikwJPVpML1/img.png](https://blog.kakaocdn.net/dn/bU7Dj6/btsD4sciZNQ/Jiyj1FM0SZThikwJPVpML1/img.png)

```bash
db.<collection>.createIndex( { <arrayField>: <sortOrder> } )
```

addr.zip 필드에 multikey index를 활용할 수 있습니다.

**검색기능을 지원하는 Text Index**

```bash
db.<collection>.createIndex(
   { <field>: "text" },
   { default_language: <language> }
)
```

mysql의 full-text index처럼 텍스트에 대한 검색을 지원하는 index입니다.

**여러 개의 필드에 인덱스를 한 번에 걸 수 있는 Wildcard Index**

```bash
db.collection.createIndex( { "<field>.$**": <sortOrder> } )
```

**이외에도 활용할 수 있는 index**

- 지리공간 좌표계에 사용할 수 있는 Geospatial Index
- hash function을 활용하는 hash index

### **인덱스의 속성**

인덱스를 사용하는 방식 및 인덱스가 저장되는 방식에 영향을 줄 수 있는 속성을 정의할 수 있습니다.

**Case-Insensitive Index**

```bash
db.createCollection("fruit")
db.fruit.createIndex( { type: 1},
                      { collation: { locale: 'en', strength: 2 } } )

db.fruit.insertMany( [
   { type: "apple" },
   { type: "Apple" },
   { type: "APPLE" }
] )

db.fruit.find( { type: "apple" } ) // does not use index, finds one result

db.fruit.find( { type: "apple" } ).collation( { locale: 'en', strength: 2 } )
// uses the index, finds three results

db.fruit.find( { type: "apple" } ).collation( { locale: 'en', strength: 1 } )
// does not use the index, finds three results
```

대소문자를 구분하지 않는 인덱스를 활용할 수 있습니다.

**Hidden Index**

```bash
db.addresses.createIndex(
   { borough: 1 },
   { hidden: true }
);
```

인덱스를 삭제하지 않고도 숨김 처리 하기

**Partial Index**

```bash
db.restaurants.createIndex(
   { cuisine: 1, name: 1 },
   { partialFilterExpression: { rating: { $gt: 5 } } }
)
```

부분 인덱스는 인덱스에 대한 하위 집합을 인덱스 지원하면서 인덱스 관리에 드는 비용을 절감할 수 있습니다.

rating이 5보다 클 경우에만 인덱스가 생성되고 인덱스의 조건인 5보다 큰 경우에 위배되는 상황에서는 인덱스를 활용하지 않습니다.

**Sparse Index**

optional filed와 같이 문서에 존재하지 않는 필드를 색인화하는데 유용합니다.

**TTL Index**

일정 시간이 지나면 컬렉션에서 문서를 자동으로 제거하는 인덱스

사용자 세션, 로그 또는 캐시 항목등을 활용할 수 있습니다.

**Unique Index**

중복된 값을 허용하지 않는 인덱스

### **기존에 존재하는 컬렉션에 인덱스 만들기**

기존에 존재하는 컬렉션에 인덱스를 만들기 위해서는 인덱스가 내부적으로 어떻게 생성되는지 이해해야 Live(Production) 레벨에 나가있는 상용 데이터베이스에 문제가 발생하지 않을 수 있습니다.

### **MongoDB 인덱스는 내부적으로 어떻게 생성될까요?**

MongoDB 4.2부터 인덱스가 생성되는 처음과 끝에 메타데이터 변경을 보호하기 위해 exclusive lock을 잡는 최적화된 빌드 프로세스를 사용합니다. (빌드 프로세스의 시자과 끝에서만 잠금이 유지되어 효율적이게 바뀌었습니다)

이전 버전의 MongoDB는 foreground 또는 background 인덱스 빌드를 지원했습니다.

foreground 인덱스는 속도가 빠르고 효율적인 인덱스 데이터구조를 생성했지만 인덱스가 생성되는 동안 데이터베이스의 모든 읽기, 쓰기 인덱스를 차단했습니다.

background 인덱스 빌드는 속도가 느리고 덜 효율적이지만 읽기, 쓰기를 허용합니다.

대용량의 데이터가 존재하는 테이블에 인덱스를 추가하기 전에는 인덱스를 생성하는데 Lock을 잡을 수 있음을 이해하고 성능 하락 또는 장애에 대비해야 합니다.

또한 대상 컬렉션에 쓰기 부하가 많은 기간에 인덱스를 빌드하면 쓰기 성능이 저하되고 인덱스 빌드 시간이 길어질 수 있습니다.

### **Unique 인덱스를 기존의 컬렉션에 추가하면 어떻게 될까요?**

만약 Unique 인덱스를 적용하려는 필드가 중복된 데이터를 가지면 오류가 발생합니다.

이때 인덱스 빌드는 성공적으로 시작되지만 빌드가 끝날 때 위반 사항이 존재한다면 빌드가 종료되고 오류가 발생합니다.

이때 Collection이 분산되어 있는 상황이라면 인덱스 생성 작업이 일부 Shard에만 성공할 수 있기 때문에 Shard 간 일관성 이 없는 인덱스가 남을 수 있습니다.

이를 방지하기 위해서는 db.collection.dropIndex()를 실행하여 컬렉션에서 인덱스를 삭제합니다.

### **MongoDB의 인덱스 빌드를 최대 몇개 허용할까요?**

기본적으로 최대 3개의 동시 인덱스 빌드를 허용합니다.

maxNumActiveUserIndexBuilds 매개변수를 수정하면 최대 허용 개수를 늘릴 수 있습니다.

### **Replicated 환경에서 인덱스 생성하기**

4.4 이상에서 복제된 환경 또는 샤딩된 클러스터 전체에서 인덱스 빌드를 동시에 수행할 수 있습니다.

Primary 노드가 커밋을 위해 정족수를 확보하면 키 제약 조건 위반이 없는지 확인하고 위반이 존재한다면 빌드는 실패합니다.

### **복제된 환경에서 Index 롤링 배포하기**

1. Secondary 하나를 중지하고 Standalone으로 실행하기
2. 실행한 인스턴스에 인덱스 생성하기
3. Replica Set의 구성원으로 다시 인스턴스 시작하기
4. 나머지 Secondaries에 대해 반복하기
5. 모든 secondareis에 새 인덱스가 존재하면 프라이머리를 다운시킨 후 복제 세트 멤버가 새 프라이머리를 선출합니다.
6. 이제 primary였던 노드에도 인덱스를 추가합니다.

### **인덱스 사용량 측정하기**

$indexStatus 집계를 활용하여 인덱스 사용에 대한 통계를 조회할 수 있습니다.

explain()를 활용하여 사용된 인덱스, 스캔한 문서 수, 쿼리 처리 시간 등 실행계획에 대한 통계를 반환할 수 있습니다.

hint()를 통하여 특정 인덱스를 사용하도록 강제할 수 있습니다.

hint()를 사용한다고 해서 MongoDB가 항상 hint를 따른다는 보장은 없으며 어제에는 최선의 선택이 데이터가 많아져서 오늘에는 최선의 선택이 아닐 수 있기 때문에 hint는 개인적으로 비선호합니다.

### **인덱싱 전략**

애플리케이션의 쿼리에 대해 깊이 이해하고 있어야 합니다.

쿼리의 종류, 읽기 대 쓰기 비율, 시스템의 메모리양 등의 여러 가지 요소를 고려해야 합니다.

프로덕션 환경에서 실행한 데이터 세트와 유사한 환경을 구사하고 어떤 구성이 가장 성능이 좋은지 확인하는 방법이 가장 좋은 전략입니다.

다양한 인덱싱 전략들에 대해서도 알아보겠습니다.

### **THE ESR(Equality, Sort, Range) Rule**

Equality는 정확히 값이 일치하는 것을 뜻합니다.

서칭 해야 하는 문서 수를 제한하기 위해 정확히 일치해야 하는 필드를 가장 먼저 배치하는 것이 좋습니다.

Sort는 정렬을 뜻합니다.

인덱스를 활용하면 정렬을 빠르게 수행할 수 있습니다.

Range는 범위검색을 뜻하며 쿼리 효율성을 높이기 위해서는 범위를 최대한 엄격하게 설정하고 등호 일치를 사용하여 스캔해야 하는 문서 수를 제한합니다.

**ESR rule을 통해 인덱스를 생성해보고, 이에 적합한 쿼리를 작성해 봅니다.**

```kotlin
db.cars.find(
   {
       manufacturer: 'Ford',//Equals
       cost: { $gt: 15000 }//Range
   } ).sort( { model: 1 } )//Sort
```

**적합한 인덱스**

```kotlin
{ manufacturer: 1, model: 1, cost: 1 }
```

### **Prefix 전략 - Query**

```kotlin
//이렇게 인덱스를 만들면
{ x: 1, y: 1, z: 1 }

//해당 쿼리들이 지원됨
{ x: 1 }
{ x: 1, y: 1 }
{ x: 1, z: 1 }

//해당 쿼리들은 지원되지 않음
{ z: 1 }

// {x: 1, z: 1 } 인덱스는 쿼리와 정렬 연산을 모두 지원하는 반면, { x: 1, y: 1, z: 1 } 인덱스는 쿼리만 지원합니다.
```

### **Prefix 전략 - Sort**

```kotlin
//이렇게 인덱스를 만들면
db.records.createIndex( { a: 1 } )
//해당 쿼리들이 지원됨
db.records.find().sort( { a: 1 } )
db.records.find().sort( { a: -1 } )

//이렇게 인덱스를 만들면
{ a: 1, b: -1 }
//해당 쿼리들이 지원됨
{ a: 1, b: -1 }
 { a: -1, b: 1 }
//해당 쿼리들은 지원되지 않음
{ a: -1, b: -1 }
{a: 1, b: 1}

//이렇게 인덱스를 만들면
db.data.createIndex( { a:1, b: 1, c: 1, d: 1 } )
//index의 prefiex들이 아래와 같음
{ a: 1 }
{ a: 1, b: 1 }
{ a: 1, b: 1, c: 1 }
```

|db.data.find( { a: 5 } ).sort( { b: 1, c: 1 } )|{ a: 1 , b: 1, c: 1 }|
|---|---|
|db.data.find( { b: 3, a: 4 } ).sort( { c: 1 } )|{ a: 1, b: 1, c: 1 }|
|db.data.find( { a: 5, b: { $lt: 3} } ).sort( { b: 1 } )|{ a: 1, b: 1 }|

equality의 경우와는 다르게 sort는 c, b 하나만 등장해도 정렬이 가능합니다.

**이런 쿼리는 인덱스를 타지 않을 수 있습니다.**

```kotlin
db.data.find( { c: 5 } ).sort( { c: 1 } )
```

equality에 c만 등장했기 때문입니다.

### **RAM의 크기에 맞는 Index 찾기**

```kotlin
db.collection.totalIndexSize()
```

인덱스의 바이트 크기를 측정할 수 있습니다.

인덱스의 작업 집합은 동시에 메모리에 들어갈 수 있어야 디스크를 활용하지 않습니다.

### **Cardinality가 높은 곳에 인덱스 활용하기**

그렇다(1), 그렇지 않다(0)과 같은 비트연산에서 인덱스를 활용하면 절만이 모두 Scan 되기 때문에 비효율적입니다.

반면에 ID와 같이 Cardinality가 높은 곳에 인덱스를 활용하면 1건의 문서만 Scan 될 수 있습니다.

공식문서에서는 Ensure Selectivity로 선택성을 보장하라는 의미로 활용합니다.