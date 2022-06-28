# 오늘의창피한삽질(feat.get메소드와post메소드).md
**2022. 06. 28. (화) 기록**

길게 쓸 필요도 없다... ^^

### 오늘의 역대급 5시간 삽질 한 줄 요약: **get 메소드와 post 메소드를 제대로 구분하자!**

구구절절 설명:

검색필터 기능을 구현하고 싶어서 request body로써 `params`에 json 객체(`searchFilter`라고 이름지었다)를 담아 보내려 했고, (다시 보니까 이것부터 문제였음. json객체는 body에 걍 바로 넣어야지 axios의 params로 넣는게 아니다,, ,)

그걸 넘겨받은 백엔드에서 `@RequestBody SearchFilter filter` 객체를 로그에 찍어봤는데 자꾸 .... 자꾸만 `null`만 찍히는 것이었다....

이상한건 포스트맨 테스트에서는 아주 멀쩡하게 나온다는 사실. (근데 포스트맨에서는 왜 잘 찍혔지? 이상하다... 낼 교수님께 질문)

![image](https://user-images.githubusercontent.com/18097984/176190441-be7f0571-671f-45b6-a3d0-18e166589cde.png)

자 문제의 코드는 아래와 같다.

```javascript
...
    const response = await axios.get(`/product/list?pageNo=${pageNo}&pageSize=${pageSize}`, {  // 보이다시피 문제는 여기였다... ^^ get()을 쓰고 앉았다
      params: {  // 그리고 이렇게 params를 쓰고 앉았다.
        prodType: null,
        isDisplayed: null,
        itemStatus: null,
        keywordSearchCriteria: null,
        keywordSearchKeyword: null,
        partnerCompanyId: null,
        brandId: null,
        isSoldOut: null,
        categoryId: null,
        periodSearchCriteria: null,
        periodSearchPeriod: null,
      },
    });
...
```

아래처럼 `get()`을 `post()`로 바꾸고 params가 아닌 그냥 객체로 넘기니 아주 자알 동작한다. ㅎㅎㅎㅎㅎ

```javascript
...
    const response = await axios.post(`/product/list?pageNo=${pageNo}&pageSize=${pageSize}`, {
    prodType: null,
    isDisplayed: null,
    itemStatus: null,
    keywordSearchCriteria: null,
    keywordSearchKeyword: null,
    partnerCompanyId: null,
    brandId: null,
    isSoldOut: null,
    categoryId: null,
    periodSearchCriteria: null,
    periodSearchPeriod: null,
    });
...
```
