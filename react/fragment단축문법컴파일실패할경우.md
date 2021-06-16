# 컴포넌트의 fragment 단축문법(`<> </>`)에서 컴파일 실패할 경우 해결 방법

킹받아 죽는 줄 알았다;;

![image](https://user-images.githubusercontent.com/18097984/122169882-e79c0c00-ceb8-11eb-8b3f-4ed7b3ba8478.png)
*(지금은 고쳐서 캡쳐 못하는 바람에 남의 에러를 가져온 것이긴 하지만, 이것과 똑같은 형태였음)*


create-react-app 활용해서 만든 프로젝트를 npm start했는데,

당장 어제까지만 해도 멀쩡히 잘 작동하던 사이트가 Failed to compile 에러가 뜨면서

컴포넌트에서 fragment 단축문법 `<> </>`를 syntax error로 인식하는 것이었다. unexpected token이라고...

왜지?? 그동안 멀쩡히 잘 작동하다가...

열심히 삽질하다가 알게된 결론은, 원인은 create-react-app의 버전 차이였다는 것이다;

`package.json` 파일로 들어가서 보니 `react-scripts`의 버전이 옛날 버전이었기 때문에 fragment syntax를 support하는 게 stable하지가 않았던 것 같다.

![image](https://user-images.githubusercontent.com/18097984/122170881-0cdd4a00-ceba-11eb-9f29-358ab60f0203.png)
*오류가 발생할 때 package.json 파일에서 확인한 react-scripts 버전*

(참고로 `react-scripts`는 create-react-app에 포함된 script 모음이란다. CRA의 버전을 업데이트하고 싶으면 `react-scripts`를 업데이트하면 되는 것으로 보인다.)

버전이 최소 2.0 이상은 돼야 정상적인 fragment syntax support를 해주는 것 같다.

![image](https://user-images.githubusercontent.com/18097984/122172489-bffa7300-cebb-11eb-8032-d7a79e66a0b4.png)

공식 블로그에서 설명해주는대로 우선 2.0.3 버전으로 직접 고쳐주고 npm install를 실행시킨 뒤, 다시 npm start하니 에러 없이 정상적으로 작동한다!!

2018년도 글을 참고한거긴 해서, 꼭 위에서 말한 버전으로 할 필요는 없는 것 같고 latest 버전으로 업그레이드해줘도 좋을 듯하다.


### 참고
* [React fragment shorthand failing to compile](https://stackoverflow.com/questions/48316365/react-fragment-shorthand-failing-to-compile)
* [[리액트공식블로그]Updating a Project to Create React App 2.0](https://reactjs.org/blog/2018/10/01/create-react-app-v2.html#updating-a-project-to-create-react-app-20)
