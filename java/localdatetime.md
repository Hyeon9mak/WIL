# LocalDateTime, JsonParser는 단순 문자열을 날짜, 시간 등의 형태로 변환할 뿐이다.
LocalDateTime, JsonParser 를 통해서 만들어진 LocalDateTime 개념에 헷갈릴 수 있다.
그러나 LocalDateTime은 그냥 날짜, 시간 형태를 가졌을 뿐 실제로 날짜, 시간 관련 조작(+-시간)은 아무것도 하지 못한다.
'현지시간;이라는 이름을 갖고 있기 때문에 UTC 시간 등이 넘어오면 알아서 변환해줄 것 같지만, 
실제로는 문자열에서 시간처럼 보이는 정보들만 빼어내서 가져올 뿐이다.
