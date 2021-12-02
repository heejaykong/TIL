# 이미지의exif데이터란(사진회전문제)

일을 하다 버그리포트가 들어왔다.

로딩되는 데이터 이미지의 방향이 작업하는 사람마다 다르게 보이는 문제였다.

몇몇 작업자는 이미지가 가로로 보이고, 몇몇 사람들은 이미지가 세로로 보여서

어노테이션을 하는데 네모 영역으로 태그를 했던 내역이 다르게 보이는 것이었다.(모두 크롬으로 작업하신다)

혹시 이미지를 그려주는 기존 코드에 브라우저 크기에 따라 이미지를 자동회전시켜주는 로직이 포함되어있나 확인해봤더니 그런 건 아니었다.

CSS 문제인가 싶어서 css image shows rotated <- 대충 이런 키워드로 구글링 해보니,

결론적으로 CSS의 `image-orientation: from-image;`이라는 property가 핵심인 것 같았다.

```css
img {
    image-orientation: from-image;
}
```

이미지 파일에 포함된 EXIF라는 종류의 데이터에 사진의 orientation(방향)을 지정하는 metadata가 있다고 한다.

사람이 카메라나 스마트폰으로 사진을 찍을 때 상황에 따라 디바이스를 기울여 찍을 수도 있잖는가?

그런 사진일 경우, 사진의 데이터에 꼬리표처럼

"이 사진은 찍을때 기울여 찍은 거라 지금 모습에서 90도 시계방향으로 회전해야 비로소 원본과 동일함" 과 같은 내용이 담겨져 있다. (맞겠지..? 확인 필요)

컴퓨터나 브라우저에서는 아마 이 꼬리표 데이터를 보고 적용해서 사람이 봤을 때 어색하지 않게 사진을 자동 rotate시켜서 띄워주는 것으로 보인다.

<img width="600" alt="screenshot" src="https://user-images.githubusercontent.com/18097984/144388952-04a7cbe3-b869-427d-a61a-836177e2cb41.png">

_위 이미지를 보면 상세정보에 방향: 8(90도 시계방향으로 회전함)이라고 되어있다._

`image-orientation: from-image;`에서 `from-image`는 이미지를 그릴 때, 이런 orientation(방향) 데이터를 똑같이 적용해주는 값이며, 따로 지정해주지 않아도 작용하는 **디폴트** 값이다.

그런데 이 `image-orientation` property가 모든 브라우저 버전에 지원이 되지는 않고, 크롬 같은 경우에는 81 버전부터 지원이 된다.

작업하시는 분들의 크롬 버전이 81보다 낮았기 때문에 발생한 문제가 아닐까 싶어 크롬 업데이트를 권해드린 상태다.

### 참고
* [img tag displays wrong orientation](https://stackoverflow.com/questions/24658365/img-tag-displays-wrong-orientation)
* [[MDN] image-orientation](https://developer.mozilla.org/en-US/docs/Web/CSS/image-orientation)
* [13/ Image 업로드 시 회전에 대하여 (feat. exif 메타데이터 - Orientation)](https://feel5ny.github.io/2018/08/06/JS_13/)
* [[JavaScript] 휴대폰 사진 업로드시 회전 방지](https://wickedmagica.tistory.com/239)
* [세로로 긴 이미지를 브라우저 주소창에서 열면 정상적으로 보여지는 이유?](https://medium.com/@jongmoon.yoon/%EC%84%B8%EB%A1%9C%EB%A1%9C-%EA%B8%B4-%EC%9D%B4%EB%AF%B8%EC%A7%80%EB%A5%BC-%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EC%A3%BC%EC%86%8C%EC%B0%BD%EC%97%90%EC%84%9C-%EC%97%B4%EB%A9%B4-%EC%A0%95%EC%83%81%EC%A0%81%EC%9C%BC%EB%A1%9C-%EB%B3%B4%EC%97%AC%EC%A7%80%EB%8A%94-%EC%9D%B4%EC%9C%A0-a39a4b20ca1b)
