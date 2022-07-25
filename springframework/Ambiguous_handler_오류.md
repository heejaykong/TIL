# Ambiguous_handler_오류

**2022. 07. 25. (월) 기록**

오류 났던 이유: 겹치는 경로가 있었기 때문. 하나의 URL을 여러개의 콘트롤러에서 매핑할 수 없다. 중복되는 url이 없도록 하나의 콘트롤러에서만 사용한다.

### 참고
* [Ambiguous handler methods mapped for HTTP path 오류](https://kimsg.tistory.com/39)
