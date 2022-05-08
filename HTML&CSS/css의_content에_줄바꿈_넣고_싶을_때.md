**2022. 05. 08. (일)**

# css의_content에_줄바꿈_넣고_싶을_때.md

치과 진료를 예약한 목록을 보여주는 화면에서, 어떤 예약이 '확정 대기 중'인지 '확정'인지 스탬프처럼 찍어내는 마크업을 구현하고 있었다.

다만 `확정 대기 중` << 이 문자열이 길어서 사이에 줄바꿈을 넣고 싶었다. `확정<br/>대기 중` 이렇게.

굳이 이거 하나 때문에 js를 덧붙이고 싶진 않았고 css안에서 `:before`의 `content` 속성으로 해결하고 싶었다.

찾아보니 `\A`나 `\00000a`를 사용하면 된다더라. 단! `white-space: pre;`(또는 `white-space: pre-wrap;`)을 함께 사용해야 한다.

```css
.stamp-pending > .stamp-message:before {
	content: '확정 \00000a 대기 중';
	white-space: pre;
}
```

**결과:**

<img src="https://user-images.githubusercontent.com/18097984/167290567-c10f53be-d6c5-4815-9ead-6a64fd632ebf.png" width="40%" alt="screenshot" />


### 참고
* [Add line break to ::after or ::before pseudo-element content](https://stackoverflow.com/questions/17047694/add-line-break-to-after-or-before-pseudo-element-content)
