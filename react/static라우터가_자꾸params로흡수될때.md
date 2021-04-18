# static 라우터가 params를 사용한 라우터로 잘못 인식되는 경우

**2021/04/18 기록**

ClubManager 플젝을 하던 중, 라우터를 설정하는데...

자꾸 static한 `path="/web"` 라우터가, 클럽이름을 parameter로 갖는 `path="/:clubName"` 라우터마냥 뜬금없이 인식돼서 화면 전환이 맘처럼 안 됐다

이유는 `<Switch>` 컴포넌트를 사용하지 않았기 때문이었고, 순서를 잘못 놨기 때문이었다.

참고한 해결 방법: Use a Switch (which stops after the first match) and place the fixed path first. -> 즉 static한 애들이 먼저 적혀야 한다.

### 참고
* [How to set the params react router not misunderstand with the static router?](https://stackoverflow.com/questions/51110111/how-to-set-the-params-react-router-not-misunderstand-with-the-static-router)
