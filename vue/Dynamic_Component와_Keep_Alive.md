# Dynamic Component와 KeepAlive

**2022. 09. 01. (목) 기록**


탭 기능을 구사하는 화면을 구현할 때처럼,

어떤 인풋값에 따라 화면이 그때그때 switch되어야 하는 로직을 짜기 위해서는

다음과 같이 vue.js에서 제공하는 `component`라는 컴포넌트를 사용할 수 있다,
``` vue
<!-- Component changes when currentTab changes -->
<component :is="tabs[currentTab]"></component>
```

`component` 태그는 `is` attribute를 통해 어떤 컴포넌트를 랜더링해줄지 동적으로 지정할 수 있는 태그다.

근데 이 `component` 태그는 화면이 switch될 때마다 해당 컴포넌트가 mount, unmount되는 구조이기 때문에

전환 시마다 unmount되는 게 싫다면 `component` 태그 대신 `KeepAlive` 태그를 사용할 수 있다.

`KeepAlive` 태그는 동적인 화면전환이 필요할 때, 컴포넌트 instance를 캐싱해주는 기능을 제공한다.

따라서 컴포넌트를 unmount해버리는 바람에 상태값이 다 리프레시되어버리는 상황을 걱정하지 않아도 되는 것 같다.
```vue
<!-- Inactive components will be cached! -->
<KeepAlive>
  <component :is="activeComponent" />
</KeepAlive>
```
### 참고
* [Vue.js 공식문서 - Dynamic Components](https://vuejs.org/guide/essentials/component-basics.html#dynamic-components)
* [Vue.js 공식문서 - KeepAlive](https://vuejs.org/guide/built-ins/keep-alive.html)
