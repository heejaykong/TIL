# Git Fork와 Clone차이

2021/02/03 기록

남의 것을 내 repository로 복사해온다는 특징은 같되, fork는 pull request를 보낼 수 있고, clone은 보낼 수 없다.

fork한 repository든 clone한 repository든, 원본 repository에 맞춰 동기화하고 싶을 때 git 명령어에 사용할 수 있는 `upstream`이라는 키워드를 배웠다. 즉 원본을 upstream이라 부른다.

아!!! 그리고 fork한 repository인 경우에는 원본 repository와 비교했을 때 몇 단계의 commit만큼 뒤쳐졌는지 위에 안내문구를 띄워주는 것 같다.

결론적으로 친구가 판 repository에서 프로젝트를 같이 진행한 경우, 내 repository에도 똑같이 불러오고 싶으면 fork해오는 게 여러모로 편할 것으로 보인다.


### 참고

- [[Git] Fork 한 repository 최신으로 동기화하기](https://json.postype.com/post/210431)
