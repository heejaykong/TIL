# computed()를_알아보자.md

**2022. 06. 25. (토) 기록**

computed()는 반응형 또는 props 속성을 복잡하게 연산해야 하는 경우 유용하게 사용할 수 있는 메소드다.

여기서 복잡한 연산이란 다음과 같은 경우를 예시로 들 수 있다.
```vue
<span>{{ author.books.length > 0 ? 'Yes' : 'No' }}</span>
```
굳이 템플릿 안에 저렇게 매번 연산하고 싶지 않으니까, 그리고 가독성도 챙길 겸 computed() 메소드를 쓰는거다.

`computed()` 메소드를 쓴다면 다음과 같이 작성할 수 있을 것이다.

```vue
<script setup>
import { reactive, computed } from 'vue'

const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})

// a computed ref
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})
</script>

<template>
  <p>Has published books:</p>
  <span>{{ publishedBooksMessage }}</span>
</template>
```

`computed()`에 들어가는 콜백함수는 getter함수처럼 작성하면 되고, `ref` 속성을 리턴해주며, 리턴된 속성은 일반 `ref` 속성처럼 `publishedBooksMessage.value` 이런 형태로 접근할 수 있다.

또 `computed()` 메소드는 자기 안에 다뤄지는 반응형 또는 props 속성의 값이 변경될 때마다 그걸 감지하고 알아서 리턴값을 업데이트해서 바인딩해준다. 아주 편리하다~

끝.

### 참고
* [Vue.js 공식문서 - Computed Properties](https://vuejs.org/guide/essentials/computed.html)
* [Vue.js 공식문서 - Props 설명 중 computed()이 언급되는 부분](https://vuejs.org/guide/components/props.html#one-way-data-flow)
