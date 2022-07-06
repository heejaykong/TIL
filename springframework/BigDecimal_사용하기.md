# BigDecimal_사용하기.md
**2022. 07. 06. (수) 기록**

보통 금융권에서 금액을 다루는 칼럼은 데이터타입이 DECIMAL이라고 한다.

INT가 커버할 수 있는 범위만으로는 큰 금액들을 다루지 못해서 DECIMAL을 쓴다고...

이 DECIMAL에 상응하는 Java단에서의 데이터타입이 뭔지 몰라 찾아보니 BigDecimal이라는 데이터타입(클래스)가 있는 것으로 보인다.

초기화와 사칙연산이 좀 까다롭다.

### 참고
* [JAVA BigDecimal 사칙연산(더하기, 빼기, 나누기, 곱하기), 비교 compareTo, 소수점 처리 (올림, 버림, 반올림)](https://cofs.tistory.com/339)
