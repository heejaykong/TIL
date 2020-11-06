# addEventListener 함수

## 오늘 배운 핵심(2020-11-06)

addEventListener("행동", handler함수)에서 handler함수의 argument에 해당 event가 객체처럼 들어간다는 점.

e.g.

```jsx
function handleResize() {
	console.log("i am resized");
}

window.addEventListener("resize", handleResize);
```

위 코드에서 `handleResize()` 함수의 괄호 사이에는 event가 argument로써 (굳이 명시하지 않아도) 알아서 들어간다는 것.

`handleResize()` 함수를 다음과 같이 바꿔보자.

```jsx
function handleResize(event) {
	console.log(event);
}
```

handler를 위처럼 바꾸고 `window.addEventListener("resize", handleResize);` 를 때려보면 매번 창의 크기를 바꿀 때마다 콘솔에 해당 event가 객체처럼 찍히는 걸 볼 수 있음. 이미 argument에 들어가는 걸 내가 콕 찝어서 이것 좀 콘솔에 찍어줘 하고 요청한 셈.

### **참고**

노마드코더 vanillaJS 클론코딩 강의 중 [#2.4 Events and event handlers](https://nomadcoders.co/javascript-for-beginners/lectures/1468)