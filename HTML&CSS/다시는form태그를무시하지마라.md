2020/12/06 기록

진짜 눈물나

`<form>` 태그 안에는 `<form>` 태그 안에 허용되는 태그 외의 태그들은 넣지 말자.

`<form>` 안에 아무 생각 없이 `<ul>` 넣어서 그 안에 자바스크립트로 `<li>`를 createElement해서 appendChild하는 식으로 투두리스트를 그리고 있었는데

코드는 아무 문제 없는데 두번째 인풋부터는 갑자기 localStorage가 리셋되는 너무너무 요상한 버그가 계속 안 고쳐지는 것이었음

`<form>` 태그 속 `<ul>`을 바깥으로 꺼내니까 말끔히 해결됨... ... . .. HTML을 어중간하게 아는 상태에서 JS개발 공부를 시작한 탓이다.

진짜 눈물나

HTML CSS 공부 맘 잡고 언제 한번 해야겠다.. .... ......