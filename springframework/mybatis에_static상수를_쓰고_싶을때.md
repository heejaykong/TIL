# mybatis에_static상수를_쓰고_싶을때.md

**2022. 06. 30. (목)**

마이바티스로 매퍼 작성하는데 조건문에다가 어떤 값이 특정값을 가지는지 판별하고 싶었다.

그 특정값을 작성할 때 하드코딩하고 싶지 않았고, `.java` 클래스 어디엔가 `static final` 필드로 상수화해둔 변수명을 가져오고 싶었다.

다음 예시와 같이 짜면 된다... 졸리다

_item.xml(예시)_
```xml
...
	<select id="selectbyProdCodeAndCondition" resultType="item">
		SELECT * FROM item
		WHERE prod_code = #{prodCode}
		<if test="filter.itemStatus != null">
			<if test='filter.itemStatus.equals(@com.mycompany.backend.dto.SearchFilter@VALUE__ITEM_STATUS__UNREGISTERED)'> <!-- 여기 적힌것처럼 @클래스@필드명 이런식으로 적는다 -->
				AND item_status IN (1, 2, 3)
			</if>
			<if test='filter.itemStatus.equals(@com.mycompany.backend.dto.SearchFilter@VALUE__ITEM_STATUS__ON_SALE)'>
				AND approve_confirm_date IS NOT NULL
				AND curdate() BETWEEN item_sale_start AND item_sale_end
			</if>
			<if test='filter.itemStatus.equals(@com.mycompany.backend.dto.SearchFilter@VALUE__ITEM_STATUS__NOT_ON_SALE)'>
				AND approve_confirm_date IS NOT NULL
				AND curdate() &gt; item_sale_end;
			</if>
		</if>
    ...
...
```
_SearchFilter.java(예시)_
```java
package com.mycompany.backend.dto;

import lombok.Data;

@Data
public class SearchFilter {
  public static final String NAME__PROD_TYPE = "prodType";
  public static final String NAME__IS_DISPLAYED = "isDisplayed";
  public static final String NAME__ITEM_STATUS = "itemStatus";
  public static final String NAME__KEYWORD_SEARCH_CRITERIA = "keywordSearchCriteria";
  public static final String NAME__KEYWORD_SEARCH_KEYWORD = "keywordSearchKeyword";
  public static final String NAME__PARTNER_COMPANY_ID = "partnerCompanyId";
  public static final String NAME__BRAND_ID = "brandId";
  public static final String NAME__IS_SOLD_OUT = "isSoldOut";
  public static final String NAME__CATEGORY_ID = "categoryId";
  public static final String NAME__PERIOD_SEARCH_CRITERIA = "periodSearchCriteria";
  public static final String NAME__PERIOD_SEARCH_PERIOD__RANGE_START = "periodSearchRangeStart";
  public static final String NAME__PERIOD_SEARCH_PERIOD__RANGE_END = "periodSearchRangeEnd";
  public static final String NAME__PERIOD_SEARCH_PERIOD = "periodSearchPeriod";
  
  // 상품유형 라디오버튼 상수값
  public static final int VALUE__PROD_TYPE__SINGLE = 1; // 1:단품상품
  public static final int VALUE__PROD_TYPE__OPTION = 2; // 2:옵션상품
  public static final int VALUE__PROD_TYPE__BUNDLE = 3; // 3:묶음상품

  // 판매상태 라디오버튼 상수값
  public static final String VALUE__ITEM_STATUS__UNREGISTERED = "UNREGISTERED"; // UNREGISTERED:온라인미등록
  public static final String VALUE__ITEM_STATUS__ON_SALE = "ON_SALE";           // ON_SALE:판매중
  public static final String VALUE__ITEM_STATUS__NOT_ON_SALE = "NOT_ON_SALE";   // NOT_ON_SALE:판매종료
  
  // 통합검색 선택지 상수값
  public static final int VALUE__KEYWORD_SEARCH_CRITERIA__PROD_CODE = 1;  // 1:상품코드
  public static final int VALUE__KEYWORD_SEARCH_CRITERIA__PROD_NAME = 2;  // 2:상품명
  public static final int VALUE__KEYWORD_SEARCH_CRITERIA__BRAND_NAME = 3; // 3:제조사명
  
  // 기간 검색의 대상이 되는 선택지 상수값
  public static final int VALUE__PERIOD_SEARCH_CRITERIA__PROD_REGIST_DATE = 1;  // 1:상품등록일
  public static final int VALUE__PERIOD_SEARCH_CRITERIA__SALE_START_DATE = 2;  // 2:판매시작일
  public static final int VALUE__PERIOD_SEARCH_CRITERIA__SALE_END_DATE = 3;  // 3:판매종료일
  // 기간 검색의 기간 선택지 상수값
  public static final String VALUE__PERIOD_SEARCH_PERIOD__DAY = "1D";   // 1D:하루동안
  public static final String VALUE__PERIOD_SEARCH_PERIOD__WEEK = "1W";  // 1W:일주일동안
  public static final String VALUE__PERIOD_SEARCH_PERIOD__MONTH = "1M"; // 1M:한달동안
  public static final String VALUE__PERIOD_SEARCH_PERIOD__YEAR = "1Y";  // 1Y:일년동안
  
  private Integer prodType;
  private Boolean isDisplayed;
  private String itemStatus;
  private Integer keywordSearchCriteria;
  private String keywordSearchKeyword;
  private Integer partnerCompanyId;
  private Integer brandId;
  private Boolean isSoldOut;
  private Integer categoryId;
  private Integer periodSearchCriteria;
  private Integer periodSearchRangeStart;
  private Integer periodSearchRangeEnd;
  private String periodSearchPeriod;
}
```

### 참고
* [Access public static final string in mybatis sql in mapper files](https://stackoverflow.com/questions/11948115/access-public-static-final-string-in-mybatis-sql-in-mapper-files)
* [MyBatisのMapper XMLでJavaの定数を使う方法](https://qiita.com/ApplePedlar/items/12dc389cc32f3db5557a)
