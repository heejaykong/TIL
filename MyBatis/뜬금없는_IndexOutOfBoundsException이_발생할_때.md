**2024. 05. 27. 기록**

기존에 있던 DTO 클래스에 Lombok의 `@Builder` 어노테이션 하나만 추가했을 뿐인데

테스트 중 갑자기
```
Caused by: org.apache.ibatis.exceptions.PersistenceException:
### Error querying database.  Cause: java.lang.IndexOutOfBoundsException: Index 9 out of bounds for length 9
...
```
따위의 에러가 발생했다.

원인은 인자 없는 생성자가 없었기 때문이었다.

마이바티스 resultMap는 기능 특성상 사용할 인스턴스를 미리 생성하는데,

`@Builder` 어노테이션으로 인해 모든 인자가 포함된 생성자만이 존재할 경우엔 인스턴스를 미리 생성할 수 없어 예외를 발생시킨다.

이건 인자가 없는 생성자를 추가하면 해결되기에, 문제의 DTO 클래스에 `@NoArgsConstructor`를 추가해줬더니 해결됐다.


참고로 `@Builder`와 `@NoArgsConstructor` 어노테이션을 함께 사용하려면 `@AllArgsConstructor`도 함께 사용해야 한다.

```
e.g.
@Builder
@NoArgsConstructor
@AllArgsContructor
public class ExampleDTO {...}
```

### 참고
* [티스토리 - Mybatis의 IndexOutOfBoundsException?](https://lob-dev.tistory.com/35)
* [[Lombok] @Builder와 @NoArgsConstructor 함께 사용하기](https://blog.leocat.kr/notes/2018/09/02/lombok-using-builder-and-noargsconstructor-together)
