# svg 파일을 다루며 배운 것들

**2021. 07. 07 기록**

헬스장 오픈을 준비하는 지인 분의 사이트를 만들어 주기로 했다.

![image](https://user-images.githubusercontent.com/18097984/124772964-79a6aa00-df77-11eb-931c-a266b58f5c83.png)

위 사진에 빨간 화살표로 표시한 S자모양의 흰색막대를, 섹션이 구분될 때마다 divider처럼 넣어달라는 요청을 받았다.

svg는 벡터로 이루어져 있어서, 픽셀로 이루어진 다른 이미지 확장자들과는 달리 확대를 해도 깨지지 않는다길래 웬만한 이미지 소스들은 svg로 요청했다.

처음엔 그냥 이미지만 넣어서 absolute로 위치 조정하면 될 줄 알았는데, 반응형으로 만들려니 문제가 생겼다.
브라우저 width가 줄어들때마다 divider 이미지도 함께 resize되는 바람에 뒤에 가려져 있던 배경이 드러난다.(아래 사진처럼)

<img src = "https://user-images.githubusercontent.com/18097984/124774027-6ea04980-df78-11eb-96d5-78b8d971738a.png" width="60%" height="60%">

그래서 divider 이미지의 width는 100%으로 두고 height를 몇px로 고정시켜 양쪽으로 stretch하게끔 하려 했는데 안되는 대신 height의 px에 맞춰서 원본 비율 유지한 채로 줄어들기만 했다.
일반 img 태그처럼 작동하지 않는 것이다.

구글링해본 결과 svg를 부모 element의 ratio에 맞게 stretch하고 싶으면 preserveAspectRatio라는 attribute를 건드려야 하더라...

<img src = "https://user-images.githubusercontent.com/18097984/124770171-11ef5f80-df75-11eb-9588-7341424b68d0.png" width="40%" height="40%">

맨 마지막 요소처럼 svg 이미지를 stretch하고 싶으면 svg태그 안에 `preserveAspectRatio="none"`이 필요하다. 적용한 결과 아래같이 뒷배경이 드러나는 것을 방지할 수 있었다.

<img src = "https://user-images.githubusercontent.com/18097984/124779538-d3f63980-df7c-11eb-8818-20688304667c.png" width="60%" height="60%">


TMI:
사실, (구글링해서 알아낸 내 지식에 범위 내에서) 제일 좋아보이는 해결책은 디자이너에게 다시 반응형에 걸맞는 svg파일을 요청해서
`fill` property의 색상을 그 뒤에 있는 배경의 색깔에 맞게 채우는 방법인 것 같았다.(아래 **참고** 항목의 첫번째, 두번째 링크 참조)

그러나 그걸 당장 할 수는 없는 상황이라 차선책을 생각해야 했고, 그래서 우선 양쪽으로 stretch하는 방법을 생각해낸 것...

divider가 좀 못생겨질 순 있어도 우선 급한대로 반응형에 써먹을 순 있었다.

### 참고
* [23+ CSS Horizontal Divider Inspiration Examples](https://onaircode.com/html-css-horizontal-divider-examples/)
* [[위 링크 예제 중 하나] Free Collection of SVG CSS Horizontal Separator/Divider](https://codepen.io/xyzzyestudioweb/pen/JgdKOR?editors=1100)
* [[MDN] preserveAspectRatio](https://developer.mozilla.org/en-US/docs/Web/SVG/Attribute/preserveAspectRatio)
