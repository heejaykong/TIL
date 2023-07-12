# useState에_ES6_클래스를_사용하면_안_되는_이유.md

**2022. 12. 02. (금) 기록**

요약:
1. 클래스는 mutable한 데 반해, 리액트의 state는 immutable해야만 한다.
2. 애시당초 리액트가 권장하는 useState hook 사용 방식은, 관리되는 상태값들을 스칼라값으로 쪼개거나, 같은 시기에 갱신되는 비슷한 성격의 놈들끼리 최소한 묶어서 관리하는 방식이다.
3. 따라서 캡슐화를 명분으로 useState에 굳이 ES6 클래스 객체를 집어넣는 괜한 짓은 하지 말자는 결론.
4. 맘에 안들면 [useReducer](https://reactjs.org/docs/hooks-reference.html#usereducer) 또는 커스텀 hook을 작성하면 된다.

투두리스트 프로그램을 리액트로 짜다가, KanbanBoard, Task 등의 정보가 담긴 객체들을 상태로써 관리를 해줘야 했다.

functional component로 짜고 있었기 때문에 useState hook을 사용하고 있었는데,

괜히 KanbanBoard, Task 상태 정보들을 ES6 클래스 형태로 캡슐화시켜서 관리하고 싶어진 것이다.

예를 들어

```react.js
// ../data/models/KanbanBoard.js
export default class KanbanBoard {
  constructor() {
    this.tasks = ...;
    this.columns = ...;
    this.columnOrder = ...;
  }
```

```react.js
// ../routes/Home.js
import KanbanBoard from '../data/model/KanbanBoard.js';
function Home() {
  const [kanbanData, setKanbanData] = useState(new KanbanBoard());
  // 생략
}
export default Home;
```

대충 이런 식으로 말이다.

결론은 그럴 필요도, 이유도 없다는 것이었다.

이유는 다음과 같다.

1. 클래스는 mutable한 데 반해, 리액트의 state는 immutable해야만 한다.
2. 애시당초 리액트가 권장하는 useState hook 사용 방식은, 관리되는 상태값들을 스칼라값으로 쪼개거나, 같은 시기에 갱신되는 비슷한 성격의 놈들끼리 최소한 묶어서 관리하는 방식이다.
    > 공식문서에서 강조하는 내용: "...we recommend to split state into multiple state variables based on which values tend to change together."

아래 참고 목록의 첫번째 질의응답(Should Javascript ES6 Classes be Used as React State?)이 도움이 많이 됐다. 자세한 내용은 아래 링크를 자세히 읽어보자.

### 참고
* [Should Javascript ES6 Classes be Used as React State?](https://stackoverflow.com/questions/58581830/should-javascript-es6-classes-be-used-as-react-state)
* [[React 공식문서] Hooks FAQ - Should I use one or many state variables?](https://reactjs.org/docs/hooks-faq.html#should-i-use-one-or-many-state-variables)
