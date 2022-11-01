# `NaN`? `isNaN()`? `Number.isNaN()`? `Number()`? `parseInt()`? 숫자 유효성을 검사할 때 궁금했던 걸 다 부숴보자

**2022.10.31.(월) 기록**

숫자 유효성을 검사하는 기능을 구현할 때 자주 등장하는 `isNaN()`, `Number.isNaN()`, `Number()`, `parseInt()`, `parseFloat()`, ... 등 서로 비슷해보이면서도 다른 메소드들을 제대로 알고 싶어 기록한다.

순서:
1. 자바스크립트에서 `NaN`이란?
2. `isNaN()`과 `Number.isNaN()`의 차이점
3. `Number()`와 `parseInt()`의 차이점
4. 그러면 어떻게 유효성을 검증할 것인가?
---

사용자로부터 숫자를 입력 받아야 하고, 그 입력이 정수가 아닌 경우 예외처리를 하는 로직을 짜야 한다고 치자.

예를 들어 '페이지 수'를 입력 받아 처리하는 기능을 구현한다고 치자.

매개변수가 꼭 number 타입으로 입력될 거라는 보장은 없다. string 등 다양한 타입으로 입력이 들어올 수 있다는 가정 하에, `123`처럼 숫자(또는 `'123'`처럼 숫자로 변환가능한 문자열)만 되고, 다음처럼 정수가 아닌 입력은 예외처리를 해줘야 한다.
* `123.1(또는 '123.1')`
* `'123.1f'`
* `undefined`
* `null`
* `true`
* `''(empty string)`
* `' '(white space)`
* `'메롱'` 등

어떻게 구현하는 것이 가장 똑똑하고 튼튼한 로직일지 궁금했다. `Number()`, `parseInt()`, `isNaN()`, `Number.isNaN()`... 숫자 유효성을 검사할 때 자주 쓰이는 것 같은데 서로 상당히 비슷해 보이는 메소드들이 너무 많았다. 이들의 차이점을 알고 싶었다.

우선 가장 기본이 되는 `NaN`이라는 개념부터 알아보자.

## 1. 자바스크립트에서 `NaN`이란?

> The global `NaN` property is a value representing Not-A-Number.

(여기서 **Not-A-Number**는 직관적으로 '숫자가 아닌' 것을 의미하는 게 아닌, IEEE-754 standard에서 정의한 **Not-A-Number**를 의미하는 것을 알아야 한다. 즉 `isNaN`이냐고 물어보는 건, `typeof value !== 'number'`이냐고 물어보는 게 아니다! 쉽게 말하면 'you wanted me to do something impossible, so I'm reporting an error'-kind of value, 또는 'it's got a value, but that value isn't a real number'-kind of value 같은 개념이다.)

<details>
<summary><b>그래서 NaN이 정확히 뭐죠? 스탠포드 교수의 답변</b></summary>

> Q. What is a NaN value or NaN exactly (in the words of a non-math professor)?

>> A. Let's suppose you're working with real numbers - numbers like 1, π, e, -137, 6.626, etc. In the land of real numbers, there are some operations that usually can be performed, but sometimes don't have a defined result. For example, let's look at logarithms. You can take the logarithm of lots of real numbers: ln e = 1, for example, and ln 10 is about 2.3. However, mathematically, the log of a negative number isn't defined. That is, we can't take ln (-4) and get back a real number.<br/><br/>So now, let's jump to programming land. Imagine that you're writing a program that or computes the logarithm of a number, and somehow the user wants you to divide by take the logarithm of a negative number. What should happen?<br/><br/>There's lots of reasonable answers to this question. You could have the operation throw an exception, which is done in some languages like Python.<br/><br/>However, at the level of the hardware the decision that was made (by the folks who designed the IEEE-754 standard) was to give the programmer a second option. Rather than have the program crash, you can instead have the operation produce a value that means "you wanted me to do something impossible, so I'm reporting an error." The way this is done is by having the operation produce the special value NaN ("Not a Number"), indicating that, somewhere in your calculation, you tried to perform an operation that's mathematically not defined.<br/><br/>There are some advantages to this approach. In many scientific computing settings, the code performs a series of long calculations, periodically generating intermediate results that might be of interest. By having operations that aren't defined produce NaN as a result, the programmer can write code that just does the math as they want it to be done, then introduce specific spots in the code where they'll test whether the operation succeeded or not. From there, they can decide what to do. Contrast this with tripping an exception or crashing the program outright - that would mean the programmer either needs to guard every series of floating point operations that could fail or has to manually test things herself. It’s a judgment call about which option is better, which is why you can enable or disable the floating point NaN behavior.
</details>


자바스크립트에서 `NaN`의 타입은 number이다. `NaN`값은 그 어떤 숫자와도 equal하지 않다. 본인과조차도 equal하지 않다.
```javascript
typeof NaN; // 'number'
NaN === NaN; // false
```
알다시피 `NaN`값을 코드에서 직접 작성해넣을 일은 거의 없다. 대신 어떤 연산에서 뱉어내는 값이 유효하지 않은 숫자값이라는 의미를 알리는 특수기호로써 `NaN`을 활용하는 일이 훨씬 많다.

어떤 값이 `NaN`인지 판별하기 위해서는 `isNaN()` 또는 `Number.isNaN()` 메소드를 사용할 수 있다. 둘의 차이점은 뭘까?

## 2. `isNaN()`과 `Number.isNaN()`의 차이점
**요약:**
* `isNaN()`은 내부적으로 매개변수를 number로 강제변환하는데, 그렇게 해서 값이 `NaN`이 되면 `true`를 리턴한다.
* `Number.isNaN()`은 매개변수의 타입이 number이면서, 동시에 그 값이 `NaN`인 경우에만 `true`를 리턴한다.

**결론:**
* 어떤 값이 `NaN`의 의미론적 정의에 제대로 부합하는 값인지 알고 싶다면 `Number.isNaN()` 써라.

![image](https://user-images.githubusercontent.com/18097984/198996929-754b1c7d-4b9d-4d30-8260-d6a278c1cde7.png)
_뭔 소리임???????_

예시코드와 함께 봐보자
```javascript
let value = Math.log(-4); // NaN(음수의 로그값은 실수로 정의할 수 없음)
typeof value; // 'number'
isNaN(value); // true
Number.isNaN(value); // true

value = 'hahaha!'; // 그냥 문자열
typeof value; // 'string'
isNaN(value); // true (엥??? 혼란스러운 결과)
Number.isNaN(value); // false
```

앞서 설명했듯, 어떤 값이 단순히 '숫자가 아닌' 데이터타입이라고 다 `NaN`인 건 아니다. `NaN`은 전산 상 어떤 연산에 의해 유효하지 않은 **숫자값**이 발생할 때 이를 표시하는 특수값이다.

따라서 단순한 문자열인데도 `isNaN('hahaha!')`를 `true`라고 말해주는 `isNaN()`의 행위는 이상하게 보일 수밖에 없다. 왜 이러는 걸까?

**`isNaN()`은 내부적으로 매개변수의 값을 Number로 강제변환하는 로직이 내재**돼있기 때문이다. `Number('hahaha!')`는 `NaN`을 뱉어내고, 이 값이 `NaN`인지 판별하면 `true`가 나오기에 결국 `true`라는 값을 `isNaN('hahaha!')`는 뱉어내는 것이다.

<table>
<tbody>
<tr>
<th><code>x</code></th>
<th><code>Number(x)</code></th>
<th><code>isNaN(x)</code></th>
</tr>
<tr>
<td><code>undefined</code></td>
<td><code>NaN</code></td>
<td><code>true</code></td>
</tr>
<tr>
<td><code>{}</code></td>
<td><code>NaN</code></td>
<td><code>true</code></td>
</tr>
<tr>
<td><code>'hahaha!'</code></td>
<td><code>NaN</code></td>
<td><code>true</code></td>
</tr>
<tr>
<td><code>new Date('')</code></td>
<td><code>NaN</code></td>
<td><code>true</code></td>
</tr>
<tr>
<td><code>new Number(0/0)</code></td>
<td><code>NaN</code></td>
<td><code>true</code></td>
</tr>
</tbody>
</table>

위의 예시표처럼, `isNaN()`를 거쳐 `true`가 리턴되는 데이터라고 해서 `x`가 꼭 `NaN`의 정의에 부합하는 데이터라는 게 아니고, 단지 `NaN`으로 강제변환된 데이터였을 수도 있다는 것이다. `isNaN()`의 이런 행동은 혼란을 야기하기 때문에, ES6부터는 `Number.isNaN()`을 소개한다.

<table>
<tbody>
<tr>
<th><code>x</code></th>
<th><code>typeof x === 'number'</code></th>
<th><code>Number.isNaN(x)</code></th>
</tr>
<tr>
<td><code>undefined</code></td>
<td><code>false</code></td>
<td><code>false</code></td>
</tr>
<tr>
<td><code>{}</code></td>
<td><code>false</code></td>
<td><code>false</code></td>
</tr>
<tr>
<td><code>'hahaha!'</code></td>
<td><code>false</code></td>
<td><code>false</code></td>
</tr>
<tr>
<td><code>new Date('')</code></td>
<td><code>false</code></td>
<td><code>false</code></td>
</tr>
<tr>
<td><code>new Number(0/0)</code></td>
<td><code>false</code></td>
<td><code>false</code></td>
</tr>
</tbody>
</table>

위 표처럼 `Number.isNaN()`은 매개변수의 데이터타입이 number이면서 동시에 `NaN`값인지 판별해주는, "isNaN"이라는 의미론적 정의에 좀 더 걸맞는 메소드라 할 수 있겠다.

입력이 숫자인지 유효성을 판별하는 기능을 구현하기 위해 우선 `NaN`을 판별하는 메소드들을 공부해봤다. 그러면 입력값을 어떻게 숫자로 파싱해야 할까? 이제 `Number()`와 `parseInt()`의 차이를 알아볼 때가 됐다.

## 3. `Number()`와 `parseInt()`의 차이점

문자열을 숫자로 파싱할 땐 `Number()`와 `parseInt()`가 가장 흔하게 쓰이는 것을 볼 수 있었고, 이 둘의 차이점이 궁금해졌다.

**요약:**
* `Number()`은 "제대로 된 숫자"가 아닐 때 `NaN`을 리턴한다.
* `parseInt()`은 "제대로 된 숫자"가 아닐 때 "되는 데까지" 파싱한다.

**결론:**
* 상황 따라 다르겠지만 개인적으론 `Number()`를 더 자주 쓸 것 같다.

![image](https://user-images.githubusercontent.com/18097984/199207532-3fa1cd08-cea4-44d4-abe0-6bb779cb8c9f.png)
_문과는 나다._

예시코드를 보자.

```javascript
// 1.---------------
// Parsing
parseInt('32px'); // 32
parseInt('5e1'); // 5

// Convert type
Number('32px'); // NaN
Number('5e1'); // 50

// 2.---------------
parseInt(); // NaN
parseInt(null); // NaN
parseInt(true); // NaN
parseInt(''); // NaN
parseInt("\n"); // NaN
parseInt("\t"); // NaN

Number(); // 0
Number(null); // 0
Number(true); // 1
Number(''); // 0
Number("\n"); // 0
Number("\t"); // 0
```
1. 이처럼 `parseInt()`는 non-digit 문자까지 최대한 파싱하고, `Number()`는 문자열을 통째로 숫자로 변환하려는 것을 볼 수 있다.

2. 그리고 boolean 값, `undefined`, 또는 `null`처럼 예상치 못한 값이 들어온 경우엔 `parseInt()`는 `NaN`을 리턴하고, `Number()`는 `0` 또는 `1`을 리턴하는 것을 볼 수 있다.

한 가지 염두에 둘 점은 `Number()`도 `parseInt()`도, 매개변수가 "유효한" 숫자인지 검증하지는 않는다는 사실이다. (당연함.) 따라서 `Number()`나 `parseInt()` 중 하나를 사용하는 상황에서는 반드시 아래 항목들을 고려해, 적절한 메소드를 함께 써서 매개변수의 유효성을 검사해야 한다.
* Should only finite numbers be allowed? (See `Number.isFinite()`)
* Should only integers be allowed? (See `Number.isInteger()`)
* ... 등

`Number()`와 `parseInt()` 중 어떤 걸 선택할 지는 상황에 따라 다르다. 그러나 일반적으로는 `Number()`가 자주 쓰이는 것으로 보이기도 하고, `Number()`가 더 합리적으로 느껴진다. (만약 누군가가 `'1x'`를 입력했다고 해서 마치 그 사용자가 `'1'`을 친 것처럼 취급하기보다는 오류를 보여주는 것이 더 합리적일 것)

## 4. 그러면 어떻게 유효성을 검증할 것인가?
별 것도 아닌 내용들로 장황하고 길게 끌었다... `NaN`의 정의, `NaN`을 판별하는 `isNaN()`과 `Number.isNaN()`, 그리고 문자열을 숫자로 파싱하는 `Number()`와 `parseInt()`까지 살펴봤다. 그러면 이제 드디어 진짜 중요한 내용을 다뤄볼 차례다.

**정수(e.g. 쪽 수, 인원수 등) 입력 유효성 검사,,, 어떻게 할까?**

내가 짜 본 최선의 방법은 우선 다음과 같다.

```javascript
const isIntegerParsable = (n) => {
    return !Number.isNaN(parseInt(n)) && Number.isSafeInteger(Number(n));
};
```
참고로 이 로직은 아래 테스트 케이스들을 통과한다.
```javascript
// 정수 숫자
01. isIntegerParsable(-1)        => true
02. isIntegerParsable(0)         => true
03. isIntegerParsable(8e5)       => true
// 정수 문자열
04. isIntegerParsable('-1')      => true
05. isIntegerParsable('0')       => true
06. isIntegerParsable('0x89f')   => true
// 정수가 아닌 숫자
07. isIntegerParsable(123.1)     => false
08. isIntegerParsable(-3.14)     => false
// 정수가 아닌 문자열
09. isIntegerParsable('-1.5')    => false
10. isIntegerParsable('123.1f')  => false
11. isIntegerParsable('0.42')    => false
12. isIntegerParsable('.42')     => false
13. isIntegerParsable('99,999')  => false
14. isIntegerParsable('1.2.3')   => false
15. isIntegerParsable('#abcdef') => false
16. isIntegerParsable('blah')    => false
17. isIntegerParsable('메롱')    => false
// 빈문자열 또는 white space
18. isIntegerParsable('')        => false
19. isIntegerParsable(' ')       => false
20. isIntegerParsable('\t\t')    => false
21. isIntegerParsable('\n\r')    => false
// falsey 값들
22. isIntegerParsable(undefined) => false
23. isIntegerParsable(null)      => false
24. isIntegerParsable(false)     => false
```
입력값이 우선 `parseInt()` 해봤을 때 `NaN`이 나오지 않는(즉, `true`, `null`, `undefined`, `''` 등이 아닌) **제대로 된 숫자인지** 먼저 검증한 뒤, && `Number()`를 해봤을 때 **제대로 된 정수값인지** 검증하는 식의 로직이다.

`Number.isSafeInteger()`라는 아주 좋은 메소드를 발견했는데, 내부적으로 `isFinite()` 로직이 포함되어 있는, 매개변수가 안전한 정수인지 판별하는 메소드다. (자세한 건 [폴리필 참고](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Number/isSafeInteger#%ED%8F%B4%EB%A6%AC%ED%95%84))

---
## 결론
그동안 애매하게만 이해하고 있던 개념들을 제대로 정리해보니 속이 후련하다. 구구절절 길었지만 이 기록의 결론은 하나다: **입력 받은 숫자가 정수인지 유효성 검증할 땐 위에 짠 `isIntegerParsable()` 코드 갖다 쓰자**.ㅎ (물론 완벽한 로직은 아닐 수 있다. 문제 발견하면 업데이트할 예정)

끝.

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
* [[MDN 공식문서] Number.isSafeInteger()](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Number/isSafeInteger)
* [What Is NaN in JavaScript?](https://www.designcise.com/web/tutorial/what-is-nan-in-javascript)
* [What's the Difference Between isNaN() and Number.isNaN() in JavaScript?](https://www.designcise.com/web/tutorial/what-is-the-difference-between-isnan-and-number-isnan-in-javascript)
* [Number() vs parseInt()](https://thisthat.dev/number-constructor-vs-parse-int/)
* [parseFloat vs Number: What's the Difference?](https://camchenry.com/blog/parsefloat-vs-number)
* [[Stack Overflow] Validate decimal numbers in JavaScript - IsNumeric() 중 CMS가 남긴 답변](https://stackoverflow.com/a/1830844)
* [[Stack Overflow] What is NaN (Not a Number) in the words of a beginner? 중 templatetypedef(스탠포드 교수)가 남긴 답변](https://stackoverflow.com/a/59339835)
