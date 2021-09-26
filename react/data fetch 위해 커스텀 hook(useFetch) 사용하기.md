**2021. 09. 26. 기록**

# data fetch 위해 커스텀 hook(useFetch) 사용하기

json-server를 이용해 mock API를 만든 후, axios를 이용해서 데이터를 fetch하려고 했다.

useState, useEffect를 활용해 get하려고 했는데 코드가 너무 messy해지는 게 싫어서... hook을 이용해 데이터를 fetch할 때 다들 사용하는 컨벤션이 없을까 찾아봤다.

useReducer를 사용하거나, 커스텀 useFetch hook을 만들어 사용하는 방법 등이 있어 보였다. 이번에 내가 사용한 방법은 후자다.

레퍼런스 자료는 아래를 참고하면 된다.

### 참고
useFetch custom hook 방법
* [Building custom hooks in React to fetch Data](https://dev.to/shaedrizwan/building-custom-hooks-in-react-to-fetch-data-4ig6)
* [React Custom Hook - useFetch](https://dev.to/techcheck/custom-react-hook-usefetch-eid)

useReducer 방법
* [02. useReducer 로 요청 상태 관리하기](https://react.vlpt.us/integrate-api/02-useReducer.html)
