# 내 repository에 push하는데 pull request 발생했던 경험

내가 내 깃헙 저장소에 푸쉬하려는데 main이라는 브랜치 따로, master라는 브랜치 따로 보이고 자꾸만 풀 리퀘스트가 뜨는 것이 이상했던 하루... 알고보니 아래와 같았다.

## 'master'→'main' 용어 변경 때문

---

단지 기본 브랜치 이름을 'master'에서 'main'으로 바꿨기 때문에 그랬던 것..

github에서 직접 기본 브랜치의 이름을 변경할 수도 있고, 또는

`git config --global init.defaultBranch <name>`으로 git의 기본 브랜치의 이름을 바꿀 수도 있다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0ae0bb76-5c9a-41f2-b463-412d3a1fd92c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0ae0bb76-5c9a-41f2-b463-412d3a1fd92c/Untitled.png)

새로 repository 생성할 때 보이는 안내 문구도 master가 아닌 main이라고 칭하는 등 조금씩 바뀐 걸 볼 수 있다.

### 참고

- [Git의 기본 브랜치를 master에서 main으로 변경하기](https://blog.outsider.ne.kr/1503)
- [Github 기본 생성 브랜치 이름을 'master' 에서 'main' 으로 변경](https://velog.io/@modolee/github-renaming-the-default-branch-from-master-to-main)
- [[7] Git Branch](https://velog.io/@seochanh/7일차-누구나-쉽게-이해할-수-있는-Git-입문-발전편-1)