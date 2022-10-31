# `NaN`? `isNaN()`? `Number.isNaN()`? `parseInt()`? `parseFloat()`? 숫자 유효성을 검사할 때 궁금했던 걸 다 부숴보자

**2022.10.31.(월) 기록**

숫자 유효성을 검사하는 기능을 구현할 때 자주 등장하는 `Number()`, `parseInt()`, `isNaN()`, `Number.isNaN()`... 등 서로 비슷해보이면서도 다른 메소드들을 제대로 알고 싶어 기록한다.

---

사용자로부터 숫자를 입력 받아야 하고, 그 입력이 정수가 아닌 경우 예외처리를 하는 로직을 짜야 한다고 치자.

예를 들어 '페이지 수'를 입력 받아 처리하는 기능을 구현한다고 치자.

매개변수가 꼭 number 타입으로 입력될 거라는 보장은 없다. string 등 다양한 타입으로 입력이 들어올 수 있다는 가정 하에, `123`처럼 숫자(또는 `'123'`처럼 숫자로 변환가능한 문자열)만 되고, 다음처럼 정수가 아닌 입력은 예외처리를 해줘야 한다.
* `123.1(또는 '123.1')`
* `123.1f(또는 '123.1f')`
* `undefined`
* `null`
* `true`
* `''(empty string)`
* `' '(white space)`
* `'메롱'` 등

어떻게 구현하는 것이 가장 똑똑하고 튼튼한 로직일지 궁금했다. `Number()`, `parseInt()`, `isNaN()`, `Number.isNaN()`... 숫자 유효성을 검사할 때 자주 쓰이는 것 같은데 서로 상당히 비슷해 보이는 메소드들이 너무 많았다. 이들의 차이점을 알고 싶었다.

우선 가장 기본이 되는 `NaN`이라는 개념부터 알아보자.

## 자바스크립트에서 NaN이란?

> The global `NaN` property is a value representing Not-A-Number.

(여기서 Not-A-Number는 직관적으로 '숫자가 아닌' 것을 의미하는 게 아닌, IEEE-754 standard에서 정의한 Not-A-Number를 의미한다는 것을 알아야 한다.)

자바스크립트에서 `NaN`의 타입은 number이다. `NaN`값은 그 어떤 숫자와도 equal하지 않다. 본인과조차도 equal하지 않다.
```javascript
typeof NaN; // 'number'
NaN === NaN; // false
```
알다시피 `NaN`값을 코드에서 직접 작성하는 일은 거의 없다. 대신 어떤 연산에서 뱉어내는 값이 유효하지 않은 숫자값이라는 의미를 알리는 도구로써 `NaN`을 활용하는 일이 훨씬 많다.

어떤 값이 `NaN`인지 판별하기 위해서는 `isNaN()` 또는 `Number.isNaN()` 메소드를 사용할 수 있다. 둘의 차이점은 뭘까?

## `isNaN()`과 `Number.isNaN()`의 차이점
요약:
* `isNaN()`은 내부적으로 매개변수를 number로 강제변환한 뒤에, 그 값이 `NaN`이면 `true`를 리턴한다.
* `Number.isNaN()`은 매개변수의 타입이 number이면서 + 그 값이 `NaN`인 경우에만 `true`를 리턴한다.

![image](https://user-images.githubusercontent.com/18097984/198996929-754b1c7d-4b9d-4d30-8260-d6a278c1cde7.png)

_머?????????????????_

예시코드와 함께 봐보자
```javascript

```

### 참고
* [[MDN 공식문서] NaN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/NaN)
* [[MDN 공식문서] isNaN()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/isNaN)
* [[MDN 공식문서] isFinite()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/isFinite)
* [[MDN 공식문서] parseInt()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/parseInt)
* [[MDN 공식문서] parseFloat()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/parseFloat)
* [[MDN 공식문서] Number.isNaN()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/isNaN)
* [[MDN 공식문서] Number.isFinite()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/isFinite)
* [[MDN 공식문서] Number.parseInt()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/parseInt)
* [[MDN 공식문서] Number.parseFloat()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/parseFloat)
* [What Is NaN in JavaScript?](https://www.designcise.com/web/tutorial/what-is-nan-in-javascript)
* [What's the Difference Between isNaN() and Number.isNaN() in JavaScript?](https://www.designcise.com/web/tutorial/what-is-the-difference-between-isnan-and-number-isnan-in-javascript)
* [[Stack Overflow] Validate decimal numbers in JavaScript - IsNumeric() 중 CMS가 남긴 답변](https://stackoverflow.com/a/1830844)
* [[Stack Overflow] What is NaN (Not a Number) in the words of a beginner? 중 templatetypedef(스탠포드 교수)가 남긴 답변](https://stackoverflow.com/a/59339835)
