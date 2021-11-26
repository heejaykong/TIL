**2021.11.26. 기록**

# canvas의 width height

`canvas` 태그에는 직접적인 `width`와 `height` property가 있다. style 내부에서 정의하는 CSS width와 height과는 다르다.

따라서 js로 동적으로 canvas element의 크기를 조정할 때, `.width`를 바꾸는 것과 `.style.width`를 바꾸는 것은 다르다는 것을 알아두자.

```javascript
// Make a canvas that has a blurry pixelated zoom-in
// with each canvas pixel drawn showing as roughly 2x2 on screen
canvas.width  = 400;
canvas.height = 300; 
canvas.style.width  = '800px';
canvas.style.height = '600px';
```

만약 `.style.width` `.style.height`을 정의하지 않는다면 canvas element의 `.width` `.height`으로 정의한 값이 display size에 사용된다.

### 참고
* [Canvas width and height in HTML5](https://stackoverflow.com/a/4939066)
