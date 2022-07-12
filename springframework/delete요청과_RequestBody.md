# delete요청과_RequestBody.md

**2022. 07. 11. (월) 기록**

axios를 사용하는데 `await axios.delete('...', someData)` 이런 형태의 요청으로 뭔갈 Body에 담아 보내고 싶었는데, 이상하게 스프링 API에서 null값으로 넘어오는 거였다.

알고보니 delete 요청은 원래 보안 규칙상 RequestBody에 뭘 담아 보낼 수가 없는 종류의 요청인 걸로 보인다.

자세한 건 아래 참고 링크를 읽어보자.

### 참고
* [DELETE method RequestBody Error.](https://velog.io/@aszxvcb/DELETE-method-RequestBody-Error)
