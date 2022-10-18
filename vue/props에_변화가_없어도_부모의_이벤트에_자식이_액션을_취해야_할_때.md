# props에 변화가 없어도 부모의 이벤트에 자식이 액션을 취해야 할 때

**2022. 10. 18. (화) 기록**

부모의 이벤트에 따라 자식내부에서 어떤 액션을 취하고 싶을 때 고민했던 내용을 기록해본다. 구현한 코드는 맨 아래에 적었음

### 배경설명

자 내가 구현하고 싶었던 것은 뭐냐... 배경 설명을 해보면

![이해를위한도식](https://user-images.githubusercontent.com/18097984/196364239-0e0b16e5-7de0-425b-b394-4484a45574c8.png)

이와 같은 구성의 화면을 만들고 있었다.

`메인그리드` 컴포넌트 내에 `서브그리드` 컴포넌트가 중첩된 구조고,

① 서브그리드에서 선택한 아이템을,

② 추가버튼을 눌러 메인그리드로 옮기고,

③ 메인그리드에서 아이템을 하나 삭제하면 다시 서브그리드로 옮기는 기능을 지녀야 했다.

일단 **1~2번에 해당하는 서브그리드에서 메인그리드로 선택된 items들을 옮기는 기능은 `emit`으로 리스트를 넘겨주는 방식으로 구현을 잘 했다.**

### 문제점

문제는 바로 저 `??????`로 표현한 부분이다.

처음에는 당연히 vue.js가 권장하는대로,

**`props`를 자식에게 넘겨서, 자식 내부에서 그 `props`에 `watch()`를 걸어서 변화가 감지될 때마다 어떤 콜백이 fire되도록** 구현하고 싶었다.

다만 그렇게 구현했을 때 한 가지 상황을 대응하지 못하는 걸 발견했다

상상해보자

메인에 존재하는 아이템 A를 삭제했다가 -> 다시 메인에 추가했다가 -> 곧바로 다시 삭제했다고 치자.

이런 경우엔 자식 입장에서는 `props`가 변하지 않았기 때문에, `watch()`에 정의해놓은 콜백이 fire되지 않는다.

뭔말인지 모르겠으면 코드로 보자

``` vue
// 메인그리드 컴포넌트 코드
<template>
  <SubGrid :removedItem="removedItem" />
  ...
</template>

<script>
export default {
  setup(){
    const state = reactive({
      removedItem: null, // ...이하 생략
    })
    return {
      ...toRefs(state),
    };
  };
}
</script>
```

```vue
// 서브그리드 컴포넌트 코드
<script>
...
watch(() => props.removedItem,
  (newValue, oldValue) => console.log(newValue, oldValue)
);
</script>
```

대충 이렇다고 치면 **메인그리드에서 우연히 어쩌다 두 번 동일한 아이템을 넘겨주게 되면 `watch()`에서 그 `props`의 변화를 detect하지 못해서, 원하는대로 작동하지 않는다.**

(참고로 이건 `deep: true` 옵션과는 상관 없는 문제다.)

애초에 `watch()`를 사용해서는 개발할 수 없는 기능 같아 보였다.

### 내가 해결한 방법

그래서 걍 **`ref` 변수를 자식 컴포넌트에 걸어서, 자식 컴포넌트 내부의 메소드를 부모에서 직접 호출하는 방식**으로 바꿔 작성했다.

이건 컴포넌트 간 tight한 결속력을 만드는 방식이라 안 좋은 방법인 걸로 알고 있는데, 저 상황에서 달리 다른 방법이 떠오르진 않았다.

다만 이 상황을 좀 보완하고자 자식 컴포넌트 내부의 모든 메소드들이 vulnerable하게 노출되지 않도록,

애초에 밖에서 사용가능한 메소드를 자식 컴포넌트 내에서 명시함으로써 제한하는 `expose()` 기능을 이용했다.

좀 더 나은 방법을 찾게 된다면 업데이트 할 예정

### 그래서 구현한 코드

참고로 이 코드들은 Wijmo를 사용하는 코드라 별의별 props나 생소한 메소드가 많지만 맥락만 이해하면 되니까 그냥 적겠다

```vue
// 메인그리드(부모 컴포넌트)
<template>
  <Grid
    :initialized="init"
    :itemsSource="gData1"
  />
  ...생략
            <SubGrid
              @addProductsToMainGrid="addDeliveredItems"
              ref="subGrid"
            />
  ...이하생략
</template>

<script>
import { reactive, toRefs } from 'vue';

// Sub Page Components
import SubGrid from './heejayAssignmentComponents/SubGrid.vue';

export default {
  setup() {
    const addDeliveredItems = (deliveredItems) => {
      state.gData1.beginUpdate();
      deliveredItems.map((item) => {
        state.gData1.addNew(item);
        state.gData1.commitNew();
      });
      state.gData1.endUpdate();
    };

    const deliverToSubGrid = (item) => {
      state.subGrid.addProductsToThisGrid(item);
    };

    const removeFromThisGrid = (item) => {
      state.gData1.remove(item);
    };

    const removeProducts = (item) => {
      // send the target back to the grid that it came from
      deliverToSubGrid(item);
      // delete the target from current Main Tab's CollectionView too
      removeFromThisGrid(item);
    };

    /**
     * @param {FlexGrid} grid 초기화 타겟이 되는 그리드 객체
     */
    const init = (grid) => {
      // 상품목록 삭제버튼 클릭 핸들러
      grid.hostElement.addEventListener('click', ({target}) => {
        if (target.closest('.icon-delete')){
          const selectedItem = grid.selectedItems[0];
          removeProducts(selectedItem);
        }
      });
    };

    const state = reactive({
      gData1: new CollectionView([]),
      subGrid: null, // ...이하생략
    });

    return {
      init,
      ...toRefs(state),
      addDeliveredItems, // ...이하생략
    };
  },
};
</script>
```


```vue
// 서브그리드(자식 컴포넌트)
<template>
  <button @click="addProductsToMainGrid">상품추가</button>
  <Grid
    :initialized="init"
    :itemsSource="gData1
  />
  ...이하생략
</template>

<script>
import { reactive, toRefs, ref } from 'vue';

export default {
  emits: ['addProductsToMainGrid'],
  setup(props, { emit, expose }) {
    const deliverToMainGrid = (items) => {
      emit('addProductsToMainGrid', items);
    };

    const removeFromThisGrid = (items) => {
      items.map((item) => state.gData1.remove(item));
    };

    const addProductsToMainGrid = () => {
      const items = state.currentlySelectedItems;
      deliverToMainGrid(items);
      removeFromThisGrid(items);
    };

    const addProductsToThisGrid = (deliveredItem) => {
      state.gData1.beginUpdate();
      state.gData1.addNew(deliveredItem);
      state.gData1.commitNew();
      state.gData1.endUpdate();
    };

    // define exposed method when the component instance is accessed by a parent component via template refs
    expose({ addProductsToThisGrid });

    /**
     * @param {FlexGrid} grid 초기화 타겟이 되는 그리드 객체
     */
    const init = (grid) => {
      state.currentlySelectedItems = grid.selectedItems;
      // 그리드 내 선택한 아이템이 바뀔 때 핸들러
      grid.selectionChanged.addHandler((grid, e) => {
        state.currentlySelectedItems = grid.selectedItems;
      });
    };

    const state = reactive({
      gData1: new CollectionView(gridData),
      currentlySelectedItems: [],
    });

    return {
      init,
      ...toRefs(state),
      addProductsToMainGrid,
    };
  },
};
</script>
```

### 참고
* [Vue.js 공식문서 - Exposing Public Properties](https://vuejs.org/api/composition-api-setup.html#exposing-public-properties)
