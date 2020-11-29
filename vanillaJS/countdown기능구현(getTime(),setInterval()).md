# 디데이 countdown 기능 구현하기(getTime(), setInterval())

2020/11/09 기록

우선 구글링에서 복붙하지 않고 나 혼자 쌩으로 구현해보는 연습을 먼저 해봤다.

처음에 내가 직접 짠 코드

```jsx
const THE_DATE = "Dec 7, 2020 00:30:00"; //좋아하는 애니메이션 새 시즌 시작하는 날짜
const theDate = new Date(THE_DATE).getTime();
let remainder = 0;
function getCountdown(){
  const now = Date.now();
  const milliseconds = theDate - now;
  const days = Math.floor(milliseconds / (1000 * 60 * 60 * 24));
  remainder = milliseconds % (1000 * 60 * 60 * 24);
  const hours = Math.floor(remainder / (1000 * 60 * 60));
  remainder = remainder % (1000 * 60 * 60);
  const mins = Math.floor(remainder / (1000 * 60));
  remainder = remainder % (1000 * 60);
  const seconds = Math.floor(remainder / 1000);

  paintCountdown(days, hours, mins, seconds);
}

function paintCountdown(days, hours, mins, seconds){
  countdownText.innerHTML = `You have to wait for ${days}days ${hours}hours ${mins}minutes ${seconds}seconds`;
}

function init(){
  setInterval(getCountdown, 1000);
}

init();
```

힘들었다... 처음에 시간 개념을 어떻게 다뤄야 할지 감이 안 잡혔어서 몇 시간 걸렸다. 루프 돌려야 하나? 하면서 막 고민했었음.

막상 다 짜고 나니까 간단한 노가다였음

그런 다음 구글링해서 코드를 찾아봤음

```jsx
  var days = Math.floor(distance / (1000 * 60 * 60 * 24));
  var hours = Math.floor((distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
  var minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));
  var seconds = Math.floor((distance % (1000 * 60)) / 1000);
```
(참고로 위 코드에서 distance는 내 코드의 milliseconds라고 보면 된다.)
이외는 다 비슷하고 일, 시, 분, 초를 구하는 부분만 위처럼 달랐음. 내가 쓴 코드처럼 굳이 `remainder` 변수를 따로 쓰는 건 불필요했던 걸지도... 걍 이렇게 수식에 구겨 넣을 수 있구나...

더 최적화하는 방법은 없는지 궁금하다.

아 그리고, 디데이 날짜가 다 되어서 `setInterval()`함수가 도는 걸 멈추려면 `clearInterval()`를 쓰면 되는데, 그러려면 setInterval을 변수 어딘가에 넣어서(`const 변수a = setInterval{...}`) clearInterval의 argument로 넣어야 하는 거 같다. (`clearInterval(변수a)`)


2020/11/22 추가:
타이머 스타일링 방법은 아래 링크를 참고함(ul, li 태그와 inline-block 속성 사용하기)
https://codepen.io/AllThingsSmitty/pen/JJavZN


2020/11/28 추가:
공부하다가 아래와 같이 짤 수도 있음을 참고.
```jsx
function getCountdown() {
  const Dday = new Date("2020-12-07:00:30:00+0900");
  const now = new Date();
  
  const milliseconds = new Date(xmasDay - now);
  const secondsInMs = Math.floor(milliseconds / 1000);
  const minutesInMs = Math.floor(secondsInMs / 60);
  const hoursInMs = Math.floor(minutesInMs / 60);
  const days = Math.floor(hoursInMs / 24);
  const seconds = secondsInMs % 60;
  const minutes = minutesInMs % 60;
  const hours = hoursInMs % 24;
  
  const daysStr = `${days < 10 ? `0${days}` : days}d`;
  const hoursStr = `${hours < 10 ? `0${hours}` : hours}h`;
  const minutesStr = `${minutes < 10 ? `0${minutes}` : minutes}m `;
  const secondsStr = `${seconds < 10 ? `0${seconds}` : seconds}s`;
  countdownText.innerHTML = `${daysStr} ${hoursStr} ${minutesStr} ${secondsStr}`;
}
```

### 참고

- [w3schools How TO - JavaScript Countdown Timer](https://www.w3schools.com/howto/howto_js_countdown.asp)
- [How to create a countdown timer using JavaScript](http://www.educative.io/edpresso/how-to-create-a-countdown-timer-using-javascript)