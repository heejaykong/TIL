# map함수_돌리는데_null이라_에러날때.md

**2022. 07. 20. (수) 기록**

map함수를 돌리는데 막상 그 돌리고자하는 array값이 null일 경우 에러가 나는 경우가 있다.

null값일 경우 처리하는 여러 방법을 알아보자.

1.
```javascript
const asyncFunc = async () => {
  const arr = []

  testauth && testauth.map(e => {
      arr.push(e.value)
      console.log('pushed value',arr);
    })
};
```

2.
```javascript
const asyncFunc = async () => {
  const arr = testauth? testauth.map(e => e.value) : []
};
```
### 참고
* [Cannot read property of type null with async map](https://stackoverflow.com/questions/64690607/cannot-read-property-of-type-null-with-async-map)
