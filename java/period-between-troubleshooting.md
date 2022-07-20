# Period.between 트러블 슈팅

- 아이의 나이 정보는 2가지로 나눠서 표현 
  - 36개월이 초과된 시점에서는 '살' 
  - 36개월 이하는 ‘개월’ 
  - 버그가 발견된 나이대는 36개월 이하 전부 
- DB에는 아이의 생년월 정보를 갖고 있음 
- API가 호출될 때 생년월 정보를 오늘 날짜와 비교하여 나이로 바꿈 
- 비교에 `Period.between` 메서드 활용 
  - 결과를 n년 n개월 n일로 나타내어 줌 
  - 2021년 6월 1일생이면 1년 1개월 13일 
- 비교가 끝난 값에서 `months`로 개월 수를 내보냈었음 
- 13개월이 나갈 걸로 예상했으나.. 실제로는 1개월이 나갔음 
- toTotalMonths 메서드 호출하도록 변경

```kotlin
"Period.between으로 개월 수를 구할 때" - {
    val birthDate = LocalDate.of(2021, 7, 1)
    val conditionDate = LocalDate.of(2022, 7, 1)

    val between = Period.between(birthDate, conditionDate)

    "months 프로퍼티를 호출하면 년/월/일로 나눠진 월이 나온다. (0개월)" {
        between.months shouldBe 0
        between.years shouldBe 1
    }

    "toTotalMonths 함수를 호출하면 총 개월수가 나온다. (12개월)" {
        between.toTotalMonths() shouldBe 12
    }
}
```
