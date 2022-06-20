# boolean값이_씹힐때_&_CHAR(1)로_변환하고_싶을_때.md

DTO의 필드들 중 `isDisplayed`처럼 boolean 데이터타입을 갖는 필드를 mybatis로 넘겨줄 때,

이상하게 `true`, `false`값이 들어간대로 제대로 매핑되지 않는 현상을 목격했다.

찾아보니 lombok이 "is" 접두사가 붙은 필드로 생성하는 getter/setter 메소드명과, 

mybatis에서 사용하는 getter/setter 메소드명이 다른 이유로 제대로 동작하지 않던 것이었다.

데이터타입이 `boolean`였던 기존 필드들을 `Boolean`으로 변경해 해결했다.

**추가)**

MySQL 스키마에서 설계할 때는 boolean 값 칼럼의 데이터타입을 CHAR(1)로 잡고 `'Y'`, `'N'` 값을 넣는 식으로 설계했는데,
Java단에서 DB로 넘겨줄 때 매번 `true`, `false` 값을 일일이 `'Y'`, `'N'`로 바꾸는 게 싫었다.

찾아보니 mybatis단에서 커스텀으로 타입핸들러를 작성해서 적용해주면 알아서 원하는대로 타입변환을 시켜준단다.

그래서 Java의 `Boolean`과 <--> MySQL의 `CHAR(1)` 타입을 자동 파싱해주는 핸들러는 다음과 같이 작성했다.

-CustomBooleanTypeHandler.java_
```java
package com.mycompany.backend.config;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;

/*
 * Boolean: (예: true, false) <---> CHAR(1): (예: 'Y', 'N')로 서로 자동파싱할 때 쓰이는 핸들러
 * 해당 핸들러 작성 시 참고한 자료
 *  - 마이바티스 공식문서: https://mybatis.org/mybatis-3/configuration.html#typehandlers
 *  - 스택오버플로우
 *      - https://stackoverflow.com/a/13487957
 *      - https://stackoverflow.com/a/47689299
 */
public class CustomBooleanTypeHandler extends BaseTypeHandler<Boolean>{
  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, Boolean parameter, JdbcType jdbcType)
      throws SQLException {
    ps.setString(i, convert(parameter));
  }

  @Override
  public Boolean getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return convert(rs.getString(columnName));
  }

  @Override
  public Boolean getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    return convert(rs.getString(columnIndex));
  }

  @Override
  public Boolean getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    return convert(cs.getString(columnIndex));
  }
  
  private String convert(Boolean b) {
    return b ? "Y" : "N";
  }
  
  private Boolean convert(String s) {
    return "Y".equalsIgnoreCase(s);
  }
}
```

_mapper-config.xml_
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
  
<configuration>
  <!-- ...생략... -->
  <typeHandlers>
		<typeHandler jdbcType="CHAR" javaType="java.lang.Boolean" handler="com.mycompany.backend.config.CustomBooleanTypeHandler"/>
	</typeHandlers>
</configuration>
```

_product.xml_
```xml
  <!-- ...생략... -->
	<select id="selectByFilterAndPage" resultType="product">
		<!-- 필터검색 쿼리 -->
		SELECT * FROM prod
		<where>
			<if test="filter.prodType != null">
				prod_type = #{filter.prodType}
			</if>
			<if test="filter.isDisplayed != null">
				AND is_displayed = #{filter.isDisplayed, typeHandler=CustomBooleanTypeHandler} <!-- 여기!! 이렇게 타입변환이 필요한 곳에 typeHander를 지정해주면 된다-->
			</if>
			<if test="filter.partnerCompanyId != null">
				AND partner_company_id = #{filter.partnerCompanyId}
			</if>
			<if test="filter.brandId != null">
				AND brand_id = #{filter.brandId}
			</if>
		</where>
		ORDER BY prod_code
		LIMIT #{pager.startRowIndex}, #{pager.endRowNo}
	</select>
  <!-- ...생략... -->
```

### 참고
* [[Lombok] "is"prefix가 붙은 boolean, Boolean](https://ratseno.tistory.com/101)
* [[마이바티스공식문서] 매퍼 설정 - typeHanders](https://mybatis.org/mybatis-3/ko/configuration.html#typehandlers)
* [MyBatis Java Boolean to Sql enum](https://stackoverflow.com/a/13487957)
* [Want Mybatis to insert booleans as Y,N, not 0,1](https://stackoverflow.com/a/47689299)
