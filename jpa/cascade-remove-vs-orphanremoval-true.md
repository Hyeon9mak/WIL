# CascadeType.REMOVE vs orphanremoval=true

![image](https://user-images.githubusercontent.com/37354145/130393951-9b209c91-67f4-417a-8011-fd0d6943766a.png)

`POST`, `TAG` 객체가 서로 관계를 맺고 있다고 가정한다. 이 때 서로 1대N 관계다.

<br>

## 공통점 - 삭제
`CascadeType.REMOVE`와 `orphanremoval=true` 두 기능 모두 부모(ORM 개념 기준)객체가 데이터베이스에서 제거될 때 자식 객체도 함께 제거 된다.

![image](https://user-images.githubusercontent.com/37354145/130394046-95ef7e11-c1a2-44a2-b5b0-7fac34d1fd16.png)

`CascadeType.REMOVE`의 경우 삭제 속성이 전파되기 때문에, `orphanremoval=true`의 경우 `TAG`가 더 이상 참조 되지 않고 떠도는 객체(더미데이터)가 되기 때문에 삭제된다.

<br>

## 차이점 - 참조 해제
### CascadeType.REMOVE
![image](https://user-images.githubusercontent.com/37354145/130394975-95c00742-eb35-4bac-8bb7-93f09a625834.png)

`POST`에서 `TAG`에 대한 참조를 해제하더라도 별도의 데이터 삭제가 일어나지 않는다. `CascadeType.REMOVE`는 데이터베이스 레벨 개념으로 객체간 참조해제에 대해 관여하지 않는다.

### orphanremoval=true
![image](https://user-images.githubusercontent.com/37354145/130395219-3659fa19-675d-45db-901e-316474978b75.png)

`POST`에서 `TAG`에 대한 참조를 해제하면, `TAG`를 참조하는 객체가 더 이상 존재하지 않으므로 
`TAG`를 고아객체로 판단, 데이터베이스 상에서 삭제한다.

<br>

## References
- [What is the difference between CascadeType.REMOVE and orphanRemoval in JPA? - stackoverflow](https://stackoverflow.com/questions/18813341/what-is-the-difference-between-cascadetype-remove-and-orphanremoval-in-jpa)
- [JPA Cascade REMOVE vs Orphan Removal](https://circlee7.medium.com/jpa-cascade-remove-vs-orphan-removal-c246e6a76c10)
- [JPA CascadeType.REMOVE vs orphanRemoval = true](https://woowacourse.github.io/tecoble/post/2021-08-15-jpa-cascadetype-remove-vs-orphanremoval-true/)
