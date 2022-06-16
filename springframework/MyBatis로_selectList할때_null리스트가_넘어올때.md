**2022. 06. 16. (목) 기록**
# MyBatis로_selectList할때_null리스트가_넘어올때.md

품목 목록을 페이지네이션의 page 번호에 해당하는 목록을 쿼리하고 싶었다. 쿼리 매퍼는 MyBatis를 사용했다.

요청을 날릴 때 응답이 오긴 왔는데, 문제는 리스트 안에 품목객체가 담겨야 하는데 자꾸 null값이 담긴다는 것이었다.
```json
{
  "items": [null, null, null, null, null]
}
```

원인이 뭔지 검색해보니 DB에서는 칼럼명을 snake_case로 작성했는데 스프링부트에서는 DTO 필드명을 camelCase로 작성했기 때문이었다.

마이바티스에서는 매핑되는 DTO의 필드명이 칼럼명과 동일해야 한다는 걸 잊고 있었다.

일일이 필드명을 데이터베이스의 칼럼명과 동일하게 수정하긴 싫었고 다음처럼 mapUnderscoreToCamelCase 설정값을 true로 바꿨다.

_mapper-config.xml파일_
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
  
<configuration>
	<settings>
		<setting name="mapUnderscoreToCamelCase" value="true"/> <!-- 요거 -->
	</settings>
	<typeAliases>
		<typeAlias type="com.mycompany.backend.dto.Item" alias="item"/>
		<typeAlias type="com.mycompany.backend.dto.Pager" alias="pager"/>
	</typeAliases>
</configuration>
```
### 참고
* [mybatis selectList 컬럼 결과 값 null 출력](https://okky.kr/article/868682)
