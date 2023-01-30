# 글로벌 컴포넌트 동적으로 등록하는 방법(composition_API)

**2023. 01. 30. (월) 기록**

대리님께서 짜신 사이드플젝 코드를 보다가 배운 점 기록.

Vue3 Composition API를 사용중인데

자주 쓰이는 컴포넌트를 전역으로 등록하고 싶을 때

(참고로 전역 컴포넌트는... 로컬 컴포넌트와 달리 아무데서나 import문 없이 global하게 사용할 수 있는 컴포넌트를 말한다)

작성해놓은 컴포넌트를 전역으로 등록하기 위해 일일이 app.component()를 매 줄 작성하는 방식 대신에,

특정 디렉토리 아래에 있는 컴포넌트들을 한 번에 다 불러와 programmatic하게 app.component() 한 줄 딱 쓰면 끝나는 방법이 있다

구글링하니까 누가 상세하게 설명해 준 답변이 있어서 아래 참고 항목에 적어놓았으니 나중에 또 읽어보자.

### 참고
* [How to globally register a Vue 3 component with the composition API?](https://stackoverflow.com/questions/75148146/how-to-globally-register-a-vue-3-component-with-the-composition-api)
