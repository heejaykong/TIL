**2021. 10. 20. 기록**

# defer script vs. async script

`<script>` 태그에는 `defer`와 `async` 라는 attribute가 있다.

파서가 `<script>` 태그를 만나면 **하던 html 문서의 파싱은 멈추고 즉시 `<script>` 태그를 파싱하고 실행**한다.
스크립트가 외부에 있는 경우 우선 네트워크로부터 자원을 가져와야 하는데 이 또한 실시간으로 처리되고 자원을 받을 때까지 파싱은 중단된다.
만약 이 방식이 싫다면 개발자는 `<script>` 태그의 `defer(지연)` attribute을 표시할 수 있는데, 이렇게 하면 html 문서 파싱이 중간에 방해받는 일은 없게 되고, 문서 파싱이 완료된 이후에 script가 실행되는 식이다.
반면 `async(비동기)` attribute는 별도의 맥락에 의해 파싱하고 실행시키는, 비동기적으로 처리하는 속성이다.

**공통점:** 다운로드 시 페이지 렌더링을 막지 않는다

**차이점:**

<img src="https://user-images.githubusercontent.com/18097984/138105285-6ba2135c-7f93-4bdf-9385-344e0bf81a84.png" width="70%" alt="table" />

> 실무에선 defer를 DOM 전체가 필요한 스크립트나 실행 순서가 중요한 경우에 적용합니다. async는 방문자 수 카운터나 광고 관련 스크립트같이 독립적인 스크립트에 혹은 실행 순서가 중요하지 않은 경우에 적용합니다.

#### +) 좀 더 이해를 돕기 위한 그림:

<img src="https://uploads.disquscdn.com/images/4303414a00e244a653fc9a4894719b730caae6d4c647b97966b1061ae16fec48.png" width="70%" alt="example" />
<img src="https://uploads.disquscdn.com/images/84255af6a3268816f146e255c48db97c670931c351d640f844ecd78f591077a2.png" width="70%" alt="example" />

### 참고
* [브라우저는 어떻게 동작하는가?](https://d2.naver.com/helloworld/59361)
* [defer, async 스크립트](https://ko.javascript.info/script-async-defer)
