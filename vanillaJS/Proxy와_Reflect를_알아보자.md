# Proxy와_Reflect를_알아보자.md

**2022. 06. 21. (화) 기록**

남의 코드를 뜯어보던 중, 난생 처음보는 형태의 코드를 발견했다.

```javascript
const p =  new Proxy(target, {
  get(target, prop, receiver) {
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    if (prop === 'something') {
      console.error('어쩌구는 something를 설정할 수 없습니다.');
      return true;
    }
    return Reflect.set(target, prop, value, receiver);
  },
});
```
이게 무엇인고 하니... `Proxy`와 `Reflect`라는, 자바스크립트에서 기본으로 제공하는 객체였다. 아마 ES2015부터 생겼나보다.

자세한 건 친절한 아래 자료를 확인하자.

### 참고
* [JavaScript Proxy. 근데 이제 Reflect를 곁들인](https://ui.toast.com/weekly-pick/ko_20210413)
