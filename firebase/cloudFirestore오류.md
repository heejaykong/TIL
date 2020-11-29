# Cloud Firestore "Uncaught ReferenceError: firebase is not defined" 오류

2020/11/16 기록

백엔드 없이 vanilla JS로 Single page application을 만들고 있는데 좋아요 버튼을 구현하기 위해 firebase를 set up하고 있었다.

그런데 Initialize Firebase SDK into the app이 안되는 것이다.

`Uncaught ReferenceError: firebase is not defined`

또는

`Uncaught ReferenceError: Cannot access 'firebase' before initialization`

등의 오류가 계속 나서 진짜 삽질 계속했는데... 이럴 경우, 긁어온 아래 문장을 참고하자.

> Firebase JavaScript SDK: Add the script inside the index.html file before app.js script link at the bottom.

> Note: if you add the SDK script link after app.js, the app WON’T work.

눈물나... 겨우 이거 하나 때문에 하루종일 걸렸다.

### 참고

- [Build A Firebase CRUD App with Vanilla JavaScript NOW — Part 1](https://medium.com/@rajatamil/learn-to-build-firebase-crud-app-with-javascript-part-1-reading-data-b3f8f8e0d924)