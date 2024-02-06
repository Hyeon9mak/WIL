> https://www.mongodb.com/docs/manual/crud/

- mongoDB 는 RDB 기준 table = collection, record = document 구조를 이룬다.
	- 즉, collection document 를 insert 한다.

## Create Operations

(3.2 버전부터) `insertOne()`, `insertMany()` 명령어를 통해 document 를 삽입할 수 있다.

```
db.collection.insertOne()
db.collection.insertMany()

db.users.insertOne(
    {
        name: "hyeon9mak",
        status: "sleeping"
    }
)
```

MongoDB 에서 모든 CUD 동작은 atomic(document 하나 단위로 트랜잭션) 하게 동작한다.
한 번에 여러 doucment 에 영향을 주는 경우에도 document 하나 단위로 트랜잭션을 잡는다.

## Read Operations
역시나 collection 에서 document 를 찾아오는 방식. collection 에 query 를 던진다고 생각하자.

```
db.collection.find()

db.users.find(
    { age: { $gt: 18 } },      <-- 질의내용. 18세 초과. WHERE 절에 해당
    { name: 1, address: 1 }    <-- 쿼리 결과로 가져오고 싶은 내용. SELECT 절에 해당
).limit(5)                     <-- cursor
```

더 자세한 내용은 아래 문서 참고.

- [Query Documents](https://www.mongodb.com/docs/manual/tutorial/query-documents/)
- [Query on Embedded/Nested Documents](https://www.mongodb.com/docs/manual/tutorial/query-embedded-documents/)
- [Query an Array](https://www.mongodb.com/docs/manual/tutorial/query-arrays/)
- [Query an Array of Embedded Documents](https://www.mongodb.com/docs/manual/tutorial/query-array-of-documents/)

## Update Operations
collection 내부 document 들 수정.

```
db.collection.updateOne()
db.collection.updateMany()
db.collection.replaceOne()
```

역시나 atomic(document 하나 단위로 트랜잭션) 하게 동작한다.
한 번에 여러 doucment 에 영향을 주는 경우에도 document 하나 단위로 트랜잭션을 잡는다.

명시적으로 특정 document 만 업데이트도 가능하다.

```
db.users.updateMany(
    { age: { $lt: 18 } },
    { $set: { status: "reject" } }
)
```

## Delete Operations

```
db.collection.deleteOne()
db.collection.deleteMany()
```

역시나 atomic(document 하나 단위로 트랜잭션) 하게 동작한다.
한 번에 여러 doucment 에 영향을 주는 경우에도 document 하나 단위로 트랜잭션을 잡는다.

```
db.users.deleteMany(
    { status: "reject" }
)
```


## Bulk Write Operations

- MongoDB 는 bulk 연산을 지원한다.
- bulk 연산은 단일 컬렉션에 영향을 줄 수 있다.
- MongoDB 에서는 쓰기 접근권한 컨트롤이 가능하다.

```
db.collection.bulkWrite()
db.collection.insertMany()
```

## Ordered vs Unordered Operations
- 벌크 연산은 순서를 지킬수도, 아닐 수도 있다.
- 순서를 지키도록 명령을 내리면 직렬로 수행한다.
	- 연산 중 에러가 발생하면 남아있는 연산은 처리하지 않고 반환한다.
- 순서를 지키지 않는 명령을 내리면 병렬로 수행한다. (100% 는 아님)
	- 연산 중 에러가 발생해도 무시하고 나머지 연산을 수행한다.
- 일반적으로 순서를 지키는 연산이 더 느릴 수 밖에 없다.
	- 다른 연산이 입력하는 값을 기다린 후 입력을 시작할 수 있기 때문이다.
- `bulkWrite()` 는 기본적으로 순서를 지킨다. 별개로 순서를 지키지 않도록 하고 싶다면 `ordered : false` 옵션을 기입해주면 된다.

```
try {
   db.pizzas.bulkWrite( [
      { insertOne: { document: { _id: 3, type: "beef", size: "medium", price: 6 } } },
      { insertOne: { document: { _id: 4, type: "sausage", size: "large", price: 10 } } },
      { updateOne: {
         filter: { type: "cheese" },
         update: { $set: { price: 8 } }
      } },
      { deleteOne: { filter: { type: "pepperoni"} } },
      { replaceOne: {
         filter: { type: "vegan" },
         replacement: { type: "tofu", size: "small", price: 4 }
      } }
   ] )
} catch( error ) {
   print( error )
}
```


## schema validation

특정 enum만 들어가도록 하는것도 가능합니다.

```
db.createCollection("shipping", {
   validator: {
      $jsonSchema: {
         bsonType: "object",
         title: "Shipping Country Validation",
         properties: {
            country: {
               enum: [ "France", "United Kingdom", "United States" ],
               description: "Must be either France, United Kingdom, or United States"
            }
         }
      }
   }
} )
```


- findAndModify 학습
- 