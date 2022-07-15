# 재귀와_반복문(카테고리_선택하는_selext_box를 구현하며).md

**2022. 07. 15. (금) 기록**

_최종 구현한 모습_

<img src="https://user-images.githubusercontent.com/18097984/179126833-b0226011-cd1e-4189-be4e-c02e28ae8c96.png" width="80%" />

과제를 하면서 위 이미지처럼 상품의 카테고리를 선택해 필터링 검색을 할 수 있도록 select box를 구현하고 싶었다.

카테고리의 스미카는 아래처럼 순환참조로 설계된 상태였다.

카테고리 밑에 카테고리 list, 카테고리 밑에 카테고리 list .... 이렇게 계속 depth를 타는 구조이기 때문이다.

<img src="https://user-images.githubusercontent.com/18097984/179126954-8a0eed97-82b8-4af4-9adb-5400a2fcda3e.png" width="50%" />

select box의 목록 중 하나를 선택했을 때 선택되는 value는 `카테고리1` > `카테고리2` > `카테고리3` > `카테고리4` 에서 최종 depth인 `카테고리4`의 id값이어야 했다.

하지만 select box에 띄워지는 목록의 문자열은 보다시피 `카테고리1` > `카테고리2` > `카테고리3` > `카테고리4` 형태로 출력해야 했다.

(대충 고민하는 이모티콘)

재귀로 구현하자! 라는 마음을 먹은 순간

재귀는 왜인지 싫었다

왜냐

재귀는 메모리를 많이 잡아먹는다는 치명적인 단점이 있었으니까.

**재귀**와 **반복문**의 차이를 알고 싶었다. 검색해보니 좋은 자료가 있었다. (본글 맨아래 참고링크 참고하기)

날 떨어뜨린 우테코의 기술블로그였다. 눈물을 삼키고 읽어본 뒤 요약을 해보면 다음과 같다.

결론은 **성능**과 **가독성** 중 하나를 택해야 한다는 것이다.
```
**재귀**
* 장점: 코드의 가독성이 좋아진다.
* 단점: stack overflow의 가능성, 스택 프레임을 구성하고 해제하는 과정에서 반복문보다 오버헤드가 더 든다(성능 저하)

**반복문**
* 장점: 재귀보다 성능이 좋다
* 단점: 재귀보다 가독성이 좋지 않다.
```
언제나 그렇듯 모든 솔루션에는 양단의 단점을 어느정도 보완하는 제 3의 솔루션이 존재하기 마련이다.

역시나 재귀의 가독성과 반복문의 성능을 어느정도 둘 다 잡는 **꼬리재귀(tail call recursion)** 라는 기법이 있었다.

(단, `컴파일러가 꼬리 재귀 최적화를 지원해야만 실질적으로 단점을 보완할 수 있다`고 한다)

**꼬리재귀란?**

다음 코드에서 `TailRecursive` 함수처럼,

마지막 terminate되는 시점의 return문에서 결과값을 반환하고 끝내버리는 구조의 재귀함수(코드는 참고링크에서 긁어와서 Java로 적혀있다.)

일반 재귀와의 차이점은 다시 나를 호출했던 놈으로 돌아가 연산을 하고 또 돌아가 또하고 착착착 돌아갈 필요가 없다는 뜻이다.

```java
public int recursive(int n) {
  if (n == 1) {
    return 1;
  }
  return n + recursive(n - 1);
}

public int tailRecursive(int n, int acc) {
  if (n == 1) {
    return acc;
  }
  return tailRecursive(n - 1, n + acc);
}
```

꼬리재귀가 성능이 좋을 수 있는 이유는 컴파일러의 동작 방식이다. 함수가 호출되는 시점에 컴파일러는 꼬리재귀를 반복문으로 변경시키기 때문에 꼬리재귀가 최적화된다고 한다.

다시 한번 말하지만 컴파일러가 꼬리 재귀 최적화를 지원하는 컴파일러여야만 해당되는 이야기다.

결론은 꼬리재귀 사용하자.

<img src="https://user-images.githubusercontent.com/18097984/179126833-b0226011-cd1e-4189-be4e-c02e28ae8c96.png" width="80%" />

그래서 하던 얘기 마저 하자면 위처럼 depth별로 모든 카테고리를 표시하는 문자열을 출력하는 꼬리재귀함수는 다음과 같이 작성했다.
```javascript
/**
 * 작성자 : 공희재
 * 기능 : 카테고리 검색 select box에 들어갈 옵션 목록을 "카테고리1 > 카테고리2 > 카테고리3 > 카테고리4" 형태의 문자열로 변환함
 * @param {Array<Object>} totalCategories - api호출로 가져온 전체 카테고리 객체들이 담긴 배열
 * @param {Array<Object>} categoryOptions - 카테고리 select box에 들어갈 option으로써 사용될 카테고리 객체 배열
 * @param {Object} currentCategory - 현재 루핑의 대상이 되는 단일 카테고리
 * @param {String} optionString - 카테고리의 이름 문자열(예시: "**용 기구 > **용 기구 > ***용품 > ***기업")
 */
function convertCategoryOptions(totalCategories, categoryOptions = [], currentCategory, optionString = '') {
  const childrensDepth = currentCategory.depth + 1;
  const children = totalCategories.filter(
    (c) => c.depth === childrensDepth && c.upperCategoryId === currentCategory.categoryId
  );
  if (!children.length) {
    optionString = optionString + `${currentCategory.categoryName}`;
    categoryOptions.push({
      value: currentCategory.categoryId,
      name: optionString,
    });
    return;
  }
  optionString = optionString + `${currentCategory.categoryName} > `;
  for (const child of children) convertCategoryOptions(totalCategories, categoryOptions, child, optionString);
}

export { convertCategoryOptions };
```

끝

### 참고
* [반복문(iteration) vs 재귀(recursion)](https://tecoble.techcourse.co.kr/post/2020-04-30-iteration_vs_recursion/)
