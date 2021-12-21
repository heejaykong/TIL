**2021. 12. 21. 기록**

# js코드에서 django context 사용하기

string이면 따옴표와 함께, 그게 아니면 context 쓰던 방식 그대로 쓰되 `safe`와 함께 사용하면 된다.

```javascript
  const my_str = "{{ my_str }}";
  const my_dict = {{ my_dict|safe }}; // no quotes here
  const my_list = {{ my_list|safe }}; // no quotes here
```

(단, safe는 신뢰할 수 있는 데이터에만 써야 한다. 즉 유저가 submit한 데이터 등 신뢰할 수 없는 데이터는, XSS 공격을 방지하기 위해 escape를 하는 등 sanitize를 거친 뒤에 `safe`를 사용해야 한다.)

escape를 쓰는 예:

```javascript
  const context = JSON.parse("{{ context|escapejs }}");
```

### 참고
* [How to use the context variables passed from Django in javascript?](https://stackoverflow.com/questions/43305020/how-to-use-the-context-variables-passed-from-django-in-javascript/43305140)
* [Javascript에서 django ORM 객체 사용하기](https://velog.io/@neulhan/Javascript%EC%97%90%EC%84%9C-django-ORM-%EA%B0%9D%EC%B2%B4-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-feat.-kakao-%EC%A7%80%EB%8F%84-API)
