**2024. 05. 27. 기록**

TBD

이 글은 유효성 검사 로직을 구현하다 고민한 흔적... 두 가지 아젠다로 나뉜다.
1. enum 도입해 데이터 구조화
2. 스프링의 Validator 인터페이스 활용

과자 상자 안에 들어있는 과자 중, 재료가 시트러스 계열의 맛을 가졌는지를 판단하는 유효성 검증기를 작성해야 한다고 치자.

우선 다음과 같은 스키마를 가졌다고 쳐보자
```
SnackBox
ㄴ Snacks
  ㄴ Ingredients
    ㄴ Flavors
    ㄴ Country of Origin
```


우선 스프링의 Validator 인터페이스를 구현하는 구현체로 작성함

그리고 그 안에서 Flavors를 판단하는 유틸을 작성해야 함

이때 이동욱 씨의 enum 활용기 아티클을 적극 참고함,,, 내 상황과 아티글에서 다루는 구조가 비슷해서 적용하기 딱이었음 ㅎㅎ

그래서 FlavorType, FlavorTypeGroup 클래스를 각각 생성해 관계를 정의해줌

### 참고
* [우아한형제들 기술블로그 - Java Enum 활용기](https://techblog.woowahan.com/2527/)
* [우아한형제들 기술블로그 - 고르곤졸라는 되지만 고르곤 졸라는 안 돼! 배달의민족에서 금칙어를 관리하는 방법](https://techblog.woowahan.com/15764/)
* [Velog - [ 김영한 스프링 MVC 2편 - 백엔드 웹 개발 활용 기술 #4 ] 검증1 - Validation (4)](https://velog.io/@tngh4037/%EA%B9%80%EC%98%81%ED%95%9C-%EC%8A%A4%ED%94%84%EB%A7%81-MVC-2%ED%8E%B8-%EB%B0%B1%EC%97%94%EB%93%9C-%EC%9B%B9-%EA%B0%9C%EB%B0%9C-%ED%99%9C%EC%9A%A9-%EA%B8%B0%EC%88%A0-4-%EA%B2%80%EC%A6%9D1-Validation-4)
