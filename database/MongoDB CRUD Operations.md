> https://www.mongodb.com/docs/manual/crud/

- mongoDB 는 RDB 기준 table = collection, record = document 구조를 이룬다.
	- 즉, collection 에 document 를 insert 한다.

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
