**2022. 05. 06. (금) 기록**

# 금액에_쉼표를_JSTL로_찍고_싶을_때.md

팀원들이랑 JSP 미니프로젝트를 하던 중 사용자의 포인트 총액을 찍는 화면을 만들며 배운 것을 기록한다.

숫자의 세 자리마다 쉼표를 찍고 싶었는데, 자바스크립트의 `toLocaleString()`으로 굳이 화면에 script 태그를 또 작성하고 싶진 않았다.

찾아보니 JSTL의 포매터를 이용하는 방법이 있었다 ㅎㅎ

```jsp
...
  <div class="menu-btn-style-2__col">
    <span class="mypoint__amount">
      <fmt:formatNumber type="number" maxFractionDigits="3" value="${point}" />   <%-- <--- 이 부분! --%>
    </span>
    <span class="mypoint__arrow-right">
      <i class="fa-solid fa-chevron-right"></i>
    </span>
  </div>
...
```

전

<img src="https://user-images.githubusercontent.com/18097984/167056481-1844ca82-9c4e-4db4-990a-fdb0353c4ef9.png" width="30%" />

후

<img src="https://user-images.githubusercontent.com/18097984/167056698-0a9caea6-6ee0-4d3f-8c02-f08532882e24.png" width="30%" />



### 참고
* [JSTL fmt:formatNumber 금액 표시(세자리마다 쉼표)](https://treasurebear.tistory.com/43)
