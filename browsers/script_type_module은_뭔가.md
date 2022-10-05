# script_type_module은_뭔가.md
**2022. 10. 05. (수) 기록**

```html
<script type="module">
```

- 스크립트가 모듈이라는 것을 속성을 통해 명시해야 한다.
- 항상 엄격모드
- 모듈레벨 스코프가 존재함.

1. 단 한 번만 실행된다.
2. this 는 undefined다.(나는 그 누구도 섬기지 않는다)
3. 마치 'defer' 속성을 붙인 것처럼 실행된다.

여기서 잠깐... `defer`를 다시 복습해보자면
  - 외부 모듈 스크립트를 다운로드 할 때, 브라우저의 HTML 처리와 병렬적으로 불러온다.
  - HTML 처리가 완료된 후 스크립트가 실행된다.
  - 스크립트의 순서가 유지 된다.
  - 모듈 스크립트는 완전한 HTML 페이지를 볼 수 있고 문서 내 요소에 접근가능.

### 참고
* [브라우저 모듈과 ESM](https://eyabc.github.io/Doc/dev/core-javascript/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%20%EB%AA%A8%EB%93%88.html)
