## JS에서 JSON의 키와 값이 일치하는 경우 생략 가능
JSON 자료구조에 사용되는 Key와 Value가 완전히 동일한 경우 아래와 같이 생략이 가능하다.
```javascript
JSON.stringify({
    email: email,
    age: age,
    password: password
})
```
```javascript
JSON.stringify({
    email,
    age,
    password
})
```

<br>