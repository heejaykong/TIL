# VSCode로 javascript 실행이 안 돼요

Visual Studio Code로 html, css, js파일 만든 뒤 Cntrl+F5를 눌러 실행시키려는데 의도대로 되지 않고 막 launch.jason 파일을 건드려야 하는 거 같아 보였음.

결국 웹이 작동하는 원리에 대한 지식이 나에게 없어서 삽질을 한 것 같아 보인다. 결론적으론 **Live Server**라는 VSCode의 extension 프로그램을 깔아서 우선 해결했다. 지금도 완벽히 알진 못하지만 서버나 프로토콜 등에 대한 이해가 필요해 보인다.

---

참고한 영상에서 배운 내용을 적어보자면 아래와 같다.

우선, 그냥 내 로컬에서 방금 만든 .html 파일을 더블클릭해서 띄워 보면,

`file:///C:/Users/{어쩌구}/{저쩌구}/{어쩌구}/index.html`

대충 이런 형식의 주소로 뜨는 것을 볼 수 있는데, 앞의 file:///라는 것이 보여주듯 로컬에서 직접 파일을 열었다는 프로토콜로 표현되고 있다.

`http://{어쩌구}/{어쩌구}/...`

그 반면 웹서버를 통해 서비스 받는 형식의 링크는, 위처럼 우리가 흔히 아는 웹주소들처럼 앞에 http://가 붙는다.

개발할 때, 꼭 서비스 런칭을 위한 서버가 아니더라도, 문서를 실행하기 위한 개발용 웹서버가 필요하단 거다. Live Server를 설치하고 다시 실행해보니 브라우저가 뜨면서 페이지가 잘 실행됐고, 아래와 같은 주소를 보였다. 

`http://127.0.0.1:5500/{어쩌구}/index.html`

5500번이라는 포트로 해당 페이지를 서비스하는 웹서버가 존재하게 되고, http라는 프로토콜을 통해 해당 문서를 서비스했다는 의미다.

---

삽질 시간을 줄이고 싶으면 어떻게 해야 할까? 아직 구글링 하는 요령이 부족하다. 계속 삽질하다보면 감이 좀 잡히겠지? 그랬으면 좋겠다. 영어로 된 공식 문서를 읽는 것도 아직은 어색하다. 활자가 눈에 바로 들어오지 않아서 불편하다. 영어가 제1언어인 사람들이 부럽다. 이상 오늘의 일지 끝

### 참고

- [자바스크립트 & DOM for VanillaJS 프로그래밍 4강 - 코드 작성과 Live Server 설치하기](https://youtu.be/_W3LLg-Uu38)
- [How to Setup Visual Studio Code for Javascript Development 2019](https://youtu.be/THDTDTkyB1I)