**2021. 10. 21. 기록**

# `normalize.css` vs. `reset.css`

필자는 프로젝트를 시작할때 단순히 css 스타일링의 편의성을 위해, 브라우저가 제공하는 기본 스타일이 제거된 순정상태를 만들어주는 `reset.css`를 늘 적용하곤 했다.

그런데 `normalize.css`가 있다는 사실을 알게 되었고, `reset.css`과의 공통점과 차이점이 궁금해 오늘 찾아보았다. 알고보니 단순히 스타일링의 편의성 뿐만이 아니라 다음과 같이 더 중요한 역할 때문에 `reset.css` 또는 `normalize.css`를 사용한다는 사실을 배웠다.

**공통점:** 브라우저마다 기본적으로 제공하는 element 의 style 을 통일시키기 위해 사용한다. (크로스 브라우징의 역할 수행)

**차이점:**

**`reset.css` :** 기본적으로 제공되는 브라우저 스타일 **전부!! 제거**

ex) `<h1>`~`<h6>`, `<p>`, `<strong>`, `<em>` 등 웬만한 태그가 일반 text처럼 보이게 됨.

**`normalize.css` :** 기본적으로 제공되는 브라우저 스타일의 유용한 기본값은 보존한다. 브라우저 간 **일관된 스타일링**이 목표임.

ex) `<sup>` 또는 `<sub>`과 같은 태그는 `normalize.css`가 적용되어도 원래대로처럼 잘 보임.
> n<sup>2</sup> 또는 n<sub>2</sub>
> 
반면 `reset.css`를 적용하면 시각적으로 일반 텍스트와 구별 할 수 없게 됨.

> n2 또는 n2

또한 normalize.css 는 reset.css 보다 넓은 범위를 가지고 있으며 HTML5 요소의 표시 설정, 양식 요소의 글꼴 상속 부족, pre-font 크기 렌더링 수정, IE9 의 SVG 오버플로 및 iOS 의 버튼 스타일링 버그 등에 대한 이슈를 해결해준다 고...

### 참고
* [JaeYeopHan/Interview_Question_for_Beginner - FrontEnd - normalize vs reset](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/FrontEnd#normalize-vs-reset)
