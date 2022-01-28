# 재귀함수에 계속 undefined가 리턴되는 경우

**2022. 01. 28. 기록**

자바스트립트로 재귀함수를 구현하는데 이상하게 undefined만 리턴되는 것이었다.

원인은 내가 `forEach()` 문을 이용해 재귀를 짠 탓이었다.

`forEach()`의 return 값은 언제나 undefined이다. 메소드의 기본적인 속성을 몰라 실수한 케이스였다.


**추가 상식:**

`forEach()` 루프를 막을 자는 없다. 따로 exception을 던지지 않는 이상, 고작 break 따위를 이용해 `forEach()` 루프를 멈추는 방법은 없다고 한다.

즉 도중에 어떤 조건에 루프를 멈춰야 하는 로직이 필요한 경우라면, `forEach()` 메소드는 틀린 방법이다.

대신 다른 루프함수들을 사용하자

예를 들면

A simple loop

A for...of loop

Array.prototype.every()

Array.prototype.some()

Array.prototype.find()

등이 있다.

### 참고
* [Javascript recursion, foreach loop won't exit? - Anthony C의 답변](https://stackoverflow.com/a/50025649)
* [[MDN] Array.prototype.forEach()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
