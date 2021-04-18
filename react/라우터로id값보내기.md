# 리액트 라우터로 id 보내기

**2021/04/11 기록**

Club Manager 플젝에서 웹프론트앤드 개발을 맡았고, 리액트를 사용했다

기본적인 게시물 CRUD를 구현해야 하는데, 각 게시물의 id를 url에 넘겨서 띄워지게 하고 싶었음

예를 들어 `/post/${id}` 로 넘어가면 id값에 해당하는 게시글이 띄워지는 식.

이럴땐 라우터를 어떻게 만들어야 하지...? 처음 해보는거라 막막해서 *react router id* 이런 키워드들로 대충 구글링을 해보고

아래 링크를 참고해서 쉽게 할 수 있었음

나는 링크에서 제시한 두번째 방법, props를 쓰는 방식을 채택했는데(hook을 아직 안 써봐서... 나중에 써봐야지)

`match`라는 prop을 활용하면 된다.

### 참고

* [React Router, how to get data from a dynamic route](https://flaviocopes.com/react-router-data-from-route/)
