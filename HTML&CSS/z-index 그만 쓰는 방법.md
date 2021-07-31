# z-index 남발하지 않고 elements 배치시키는 방법

`position: absolute`와 `relative`를 쓰다보면 의도치 않게 element들이 겹쳐질 때가 있다.

가려져선 안되는 element가 다른 element에 가려지는 경우 등 말이다.

그럴때 게으르게 생각할 수 있는 해결책은 `z-index` 속성을 사용하는 방법일 테다. 하지만 매번 `z-index` 속성을 남발할 수는 없다.

때문에 HTML에서 element들이 근본적으로 어떤 순서대로 stack되는지 알아둘 필요가 있다.

아래 순서대로 밑에서부터 위로 쌓인다.(즉, 항목 숫자가 클수록 제일 위에 보인다)

1. root element의 배경과 border
2. `position`이 디폴트(static)인 블록들의 descendant, HTML에 나타나는 순서대로.
3. `position`이 지정된(absolute, relative 등) 블록들의 descendant, HTML에 나타나는 순서대로.

자세한 내용은 참고의 링크를 확인하자.

### 참고
* [Stacking without the z-index property](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Positioning/Understanding_z_index/Stacking_without_z-index)
