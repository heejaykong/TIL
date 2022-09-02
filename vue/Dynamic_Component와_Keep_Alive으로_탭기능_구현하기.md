# Dynamic Component와 KeepAlive으로 탭기능 구현하기

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

**그래서 실제로 탭기능을 구현한 코드는 다음과 같다.**

```vue
  <!-- ...생략... -->
        <!-- 탭 영역 -->
        <custom-flex-item fix class="container__tab">
          <custom-tab
            :items="getTabTitles()"
            :modelValue="tab.currentTabIndex"
            @update:modelValue="(newIndex) => handleTabSwitch(newIndex)"
          ></custom-tab>
        </custom-flex-item>

        <!-- 선택한 탭에 따라 바뀌는 본문 영역 -->
        <keep-alive>
          <component :is="getTabPageComponent()"></component>
        </keep-alive>
  <!-- ...생략... -->
```

```vue
<script>
// Tab Page Components
import OnlineProductCheckTabPage from './heejayAssignmentComponents/OnlineProductCheckTabPage.vue';
import NewStoreTabPage from './heejayAssignmentComponents/NewStoreTabPage.vue';
import LowSalesTabPage from './heejayAssignmentComponents/LowSalesTabPage.vue';
import OverstockTabPage from './heejayAssignmentComponents/OverstockTabPage.vue';

export default {
// setup() 함수 내 코드
// ...생략...
    const onlineProductCheckTabPage = shallowRef(OnlineProductCheckTabPage),
          newStoreTabPage = shallowRef(NewStoreTabPage),
          lowSalesTabPage = shallowRef(LowSalesTabPage),
          overstockTabPage = shallowRef(OverstockTabPage);

    const getTabTitles = () => state.tab.list.map(item => item.title);
    const handleTabSwitch = newIndex => state.tab.currentTabIndex = newIndex;
    const getTabPageComponent = () => {
      const selectedTab = state.tab.list.find(tab => tab.index === state.tab.currentTabIndex);
      return state.tab.pageComponents[selectedTab.pageComponent];
    };
    const state = reactive({
      // ...생략...
      tab: {
        list: [
          { index: 0, title: '온라인상품 조회', pageComponent: 'onlineProductCheckTabPage' },
          { index: 1, title: '신규입점', pageComponent: 'newStoreTabPage' },
          { index: 2, title: '판매부진', pageComponent: 'lowSalesTabPage' },
          { index: 3, title: '과다재고', pageComponent: 'overstockTabPage' }
        ],
        currentTabIndex: 0,
        pageComponents : {
          onlineProductCheckTabPage,
          newStoreTabPage,
          lowSalesTabPage,
          overstockTabPage
        }
      },
    });
// ...생략...
</script>
```
### 참고
* [Vue.js 공식문서 - Dynamic Components](https://vuejs.org/guide/essentials/component-basics.html#dynamic-components)
* [Vue.js 공식문서 - KeepAlive](https://vuejs.org/guide/built-ins/keep-alive.html)
