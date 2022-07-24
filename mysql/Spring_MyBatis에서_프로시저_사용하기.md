# Spring_MyBatis에서_프로시저_사용하기
**2022. 07. 24. (일) 기록**

사용한 프로시저

_1. countItems_procedure.sql_
```sql
DROP procedure IF EXISTS `countItems`;

DELIMITER $$
CREATE PROCEDURE countItems(
    IN p_prodType INT,
    IN p_displayedYn CHAR(1),
    IN p_itemStatus VARCHAR(50),
    IN p_partnerCompanyId VARCHAR(50),
    IN p_brandId VARCHAR(50),
    IN p_isSoldOut BOOLEAN,
    IN p_categoryId VARCHAR(50),
    IN p_keywordSearchKeyword VARCHAR(200),
    IN p_keywordSearchCriteria INT,
    IN p_periodSearchCriteria INT,
    IN p_periodSearchRangeStart VARCHAR(20),
    IN p_periodSearchRangeEnd VARCHAR(20)
)
BEGIN
--  파라미터들 선언
    DECLARE _prodType INT;
    DECLARE _displayedYn CHAR(1);
    DECLARE _itemStatus VARCHAR(50);
    DECLARE _partnerCompanyId VARCHAR(50);
    DECLARE _brandId VARCHAR(50);
    DECLARE _isSoldOut BOOLEAN;
    DECLARE _categoryId VARCHAR(50);
    DECLARE _keywordSearchKeyword VARCHAR(200);
    DECLARE _keywordSearchCriteria INT;
    DECLARE _periodSearchCriteria INT;
    DECLARE _periodSearchRangeStart VARCHAR(20);
    DECLARE _periodSearchRangeEnd VARCHAR(20);
    
--  상수 선언
	DECLARE VALUE__ITEM_STATUS__UNREGISTERED VARCHAR(50);
	DECLARE VALUE__ITEM_STATUS__ON_SALE VARCHAR(50);
	DECLARE VALUE__ITEM_STATUS__NOT_ON_SALE VARCHAR(50);

	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__INTEGRATED INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__PROD_CODE INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__PROD_NAME INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_NAME INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__BRAND_NAME INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_CODE INT;
    
    DECLARE VALUE__PERIOD_SEARCH_CRITERIA__PROD_REGIST_DATE INT;
    DECLARE VALUE__PERIOD_SEARCH_CRITERIA__SALE_START_DATE INT;
    DECLARE VALUE__PERIOD_SEARCH_CRITERIA__SALE_END_DATE INT;
    
    DECLARE ALONE_ITEMS_QUERY LONGTEXT;
    DECLARE BOUND_ITEMS_QUERY LONGTEXT;
    DECLARE S LONGTEXT;
    
    SET _prodType = p_prodType;
    SET _displayedYn = p_displayedYn;
    SET _itemStatus = p_itemStatus;
    SET _partnerCompanyId = p_partnerCompanyId;
    SET _brandId = p_brandId;
    SET _isSoldOut = p_isSoldOut;
    SET _categoryId = p_categoryId;
    SET _keywordSearchKeyword = p_keywordSearchKeyword;
    SET _keywordSearchCriteria = p_keywordSearchCriteria;
    SET _periodSearchCriteria = p_periodSearchCriteria;
    SET _periodSearchRangeStart = p_periodSearchRangeStart;
    SET _periodSearchRangeEnd = p_periodSearchRangeEnd;

	SET VALUE__ITEM_STATUS__UNREGISTERED = 'UNREGISTERED';
	SET VALUE__ITEM_STATUS__ON_SALE = 'ON_SALE';
	SET VALUE__ITEM_STATUS__NOT_ON_SALE = 'NOT_ON_SALE';

	SET VALUE__KEYWORD_SEARCH_CRITERIA__INTEGRATED = 1;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__PROD_CODE = 2;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__PROD_NAME = 3;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_NAME = 4;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__BRAND_NAME = 5;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_CODE = 6;
    
    SET VALUE__PERIOD_SEARCH_CRITERIA__PROD_REGIST_DATE = 1;
    SET VALUE__PERIOD_SEARCH_CRITERIA__SALE_START_DATE = 2;
    SET VALUE__PERIOD_SEARCH_CRITERIA__SALE_END_DATE = 3;


    -- 1. 묶이기 전 품목들 가져오는 쿼리 세팅하기(UNION하기 위함)
    SET ALONE_ITEMS_QUERY = '
		SELECT prod.prod_code, item.item_code, item.item_status, prod.prod_name, item.item_name, item.retail_price, item.purch_price, item.denall_price, item.monthly_goal_sale_cnt, item.annual_goal_sale_cnt, item.regist_datetime, item.approve_req_datetime, item.sale_start_date, emp.emp_name as manager_id, item.approve_confirm_datetime, item.sale_end_date, item.stock_cnt, item.item_kind, item.acc_sale_cnt, item.acc_sale_amount
		FROM item
		LEFT JOIN prod ON prod.prod_code = item.prod_code
		LEFT JOIN emp ON emp.emp_id = prod.manager_id
		WHERE prod.prod_code IS NULL ';
	IF (_prodType IS NOT NULL) THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.prod_type = ', _prodType, ' ');
	END IF;
	IF _displayedYn IS NOT NULL THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.displayed_yn = ''', _displayedYn, ''' ');
	END IF;
	IF _itemStatus IS NOT NULL THEN
		IF _itemStatus = VALUE__ITEM_STATUS__UNREGISTERED THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.item_status IN (1, 2, 3) ');
		END IF;
		IF _itemStatus = VALUE__ITEM_STATUS__ON_SALE THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.approve_confirm_datetime IS NOT NULL AND curdate() BETWEEN item.sale_start_date AND item.sale_end_date ');
		END IF;
		IF _itemStatus = VALUE__ITEM_STATUS__NOT_ON_SALE THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.approve_confirm_datetime IS NOT NULL AND curdate() > item.sale_end_date ');
		END IF;
	END IF;
    
	IF _partnerCompanyId IS NOT NULL THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.partner_company_id = ''', _partnerCompanyId, ''' ');
	END IF;

	IF _brandId IS NOT NULL THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.brand_id = ''', _brandId, ''' ');
	END IF;
	
	IF _isSoldOut IS NOT NULL THEN
		IF _isSoldOut = true THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.stock_cnt <= 0 is true ');
		END IF;
		IF _isSoldOut = false THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.stock_cnt <= 0 is false ');
		END IF;
	END IF;

	IF _categoryId IS NOT NULL THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.category_id = ''', _categoryId, ''' ');
	END IF;

	IF _keywordSearchKeyword IS NOT NULL THEN
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__INTEGRATED THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND ( MATCH (item.item_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_code) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_short_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_desc) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR prod.brand_id = (SELECT brand_id FROM brand WHERE MATCH(brand_name) AGAINST (''', _keywordSearchKeyword, ''')) ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH item.item_code AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH item.item_name AGAINST (''', _keywordSearchKeyword, ''') ');
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, ') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__PROD_CODE THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND MATCH prod.prod_code AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__PROD_NAME THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND MATCH prod.prod_name AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__BRAND_NAME THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.brand_id = (SELECT brand_id FROM brand WHERE MATCH(brand_name) AGAINST (''', _keywordSearchKeyword, ''')) ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_NAME THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND MATCH item.item_name AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
	END IF;
    SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'ORDER BY item.approve_req_datetime ');	


	-- 2. 묶인 후 품목들 가져오는 쿼리 세팅하기--
    SET BOUND_ITEMS_QUERY = '
		SELECT prod.prod_code, item.item_code, item.item_status, prod.prod_name, item.item_name, item.retail_price, item.purch_price, item.denall_price, item.monthly_goal_sale_cnt, item.annual_goal_sale_cnt, item.regist_datetime, item.approve_req_datetime, item.sale_start_date, emp.emp_id as manager_id, item.approve_confirm_datetime, item.sale_end_date, item.stock_cnt, item.item_kind, item.acc_sale_cnt, item.acc_sale_amount
		FROM item
		LEFT JOIN prod ON prod.prod_code = item.prod_code
		LEFT JOIN emp ON emp.emp_id = prod.manager_id
		WHERE prod.prod_code IS NOT NULL ';

	IF (_prodType IS NOT NULL) THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.prod_type = ', _prodType, ' ');
	END IF;
	IF _displayedYn IS NOT NULL THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.displayed_yn = ''', _displayedYn, ''' ');
	END IF;
	IF _itemStatus IS NOT NULL THEN
		IF _itemStatus = VALUE__ITEM_STATUS__UNREGISTERED THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.item_status IN (1, 2, 3) ');
		END IF;
		IF _itemStatus = VALUE__ITEM_STATUS__ON_SALE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.approve_confirm_datetime IS NOT NULL AND curdate() BETWEEN item.sale_start_date AND item.sale_end_date ');
		END IF;
		IF _itemStatus = VALUE__ITEM_STATUS__NOT_ON_SALE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.approve_confirm_datetime IS NOT NULL AND curdate() > item.sale_end_date ');
		END IF;
	END IF;
    
	IF _partnerCompanyId IS NOT NULL THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.partner_company_id = ''', _partnerCompanyId, ''' ');
	END IF;

	IF _brandId IS NOT NULL THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.brand_id = ''', _brandId, ''' ');
	END IF;
	
	IF _isSoldOut IS NOT NULL THEN
		IF _isSoldOut = true THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.stock_cnt <= 0 is true ');
		END IF;
		IF _isSoldOut = false THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.stock_cnt <= 0 is false ');
		END IF;
	END IF;

	IF _categoryId IS NOT NULL THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.category_id = ''', _categoryId, ''' ');
	END IF;

	IF _keywordSearchKeyword IS NOT NULL THEN
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__INTEGRATED THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND ( MATCH (item.item_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_code) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_short_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_desc) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR prod.brand_id = (SELECT brand_id FROM brand WHERE MATCH(brand_name) AGAINST (''', _keywordSearchKeyword, ''')) ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH item.item_code AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH item.item_name AGAINST (''', _keywordSearchKeyword, ''') ');
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, ') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__PROD_CODE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND MATCH prod.prod_code AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__PROD_NAME THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND MATCH prod.prod_name AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__BRAND_NAME THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.brand_id = (SELECT brand_id FROM brand WHERE MATCH(brand_name) AGAINST (''', _keywordSearchKeyword, ''')) ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_NAME THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND MATCH item.item_name AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
	END IF;
    
	IF _periodSearchCriteria IS NOT NULL THEN
		IF _periodSearchCriteria = VALUE__PERIOD_SEARCH_CRITERIA__PROD_REGIST_DATE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.approve_confirm_datetime BETWEEN ''', _periodSearchRangeStart, ''' AND ''', _periodSearchRangeEnd, ''' ');
		END IF;
		IF _periodSearchCriteria = VALUE__PERIOD_SEARCH_CRITERIA__SALE_START_DATE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.sale_start_date BETWEEN ''', _periodSearchRangeStart, ''' AND ''', _periodSearchRangeEnd, ''' ');
		END IF;
		IF _periodSearchCriteria = VALUE__PERIOD_SEARCH_CRITERIA__SALE_END_DATE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.sale_end_date BETWEEN ''', _periodSearchRangeStart, ''' AND ''', _periodSearchRangeEnd, ''' ');
		END IF;
	END IF;

	SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'GROUP BY prod.prod_code ');
    SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'ORDER BY item.approve_req_datetime ');
    
    -- ----------------
    SET S = 'SELECT COUNT(*) FROM (';
    
	SET S = CONCAT(S, '(', BOUND_ITEMS_QUERY,')'); -- 묶인 후 쿼리
	IF (_periodSearchCriteria IS NULL) THEN -- 사실 묶이기 전 쿼리는 이 조건일때만 UNION해야 함
		SET S = CONCAT(S, ' UNION (', ALONE_ITEMS_QUERY, ')');
	END IF;
    
    SET S = CONCAT(S, ') X;');
    -- ----------------
	SET @S = S;
    PREPARE stmt FROM @S;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END $$
DELIMITER ;

-- 아래는 테스트용도로 호출하는 쿼리
-- CALL countItems(
-- NULL --     _prodType INT,
-- ,NULL --     _displayedYn CHAR(1),
-- ,NULL --     _itemStatus INT,
-- ,NULL --     _partnerCompanyId VARCHAR(50),
-- ,NULL --     _brandId VARCHAR(50),
-- ,NULL --     _isSoldOut BOOLEAN,
-- ,NULL --     _categoryId VARCHAR(50),
-- ,NULL --     _keywordSearchKeyword VARCHAR(200),
-- ,NULL --     _keywordSearchCriteria INT,
-- ,NULL --     _periodSearchCriteria INT,
-- ,NULL --     _periodSearchRangeStart VARCHAR(20),
-- ,NULL --     _periodSearchRangeEnd VARCHAR(20),
-- );

```


_2. getItems_procedure.sql_
```sql
DROP procedure IF EXISTS `getItems`;

DELIMITER $$
CREATE PROCEDURE getItems(
	IN p_isSearchingForItem BOOLEAN,
    IN p_prodType INT,
    IN p_displayedYn CHAR(1),
    IN p_itemStatus VARCHAR(50),
    IN p_partnerCompanyId VARCHAR(50),
    IN p_brandId VARCHAR(50),
    IN p_isSoldOut BOOLEAN,
    IN p_categoryId VARCHAR(50),
    IN p_keywordSearchKeyword VARCHAR(200),
    IN p_keywordSearchCriteria INT,
    IN p_periodSearchCriteria INT,
    IN p_periodSearchRangeStart VARCHAR(20),
    IN p_periodSearchRangeEnd VARCHAR(20),
    IN p_prodCode VARCHAR(50),
    IN p_itemCode VARCHAR(50),
    IN p_rowsPerPage INT,
    IN p_startRowIndex INT
)
BEGIN
--  파라미터들 선언
	DECLARE _isSearchingForItem BOOLEAN;
    DECLARE _prodType INT;
    DECLARE _displayedYn CHAR(1);
    DECLARE _itemStatus VARCHAR(50);
    DECLARE _partnerCompanyId VARCHAR(50);
    DECLARE _brandId VARCHAR(50);
    DECLARE _isSoldOut BOOLEAN;
    DECLARE _categoryId VARCHAR(50);
    DECLARE _keywordSearchKeyword VARCHAR(200);
    DECLARE _keywordSearchCriteria INT;
    DECLARE _periodSearchCriteria INT;
    DECLARE _periodSearchRangeStart VARCHAR(20);
    DECLARE _periodSearchRangeEnd VARCHAR(20);
    DECLARE _prodCode VARCHAR(50);
    DECLARE _itemCode VARCHAR(50);
    DECLARE _rowsPerPage INT;
    DECLARE _startRowIndex INT;
    
--  상수 선언
	DECLARE VALUE__ITEM_STATUS__UNREGISTERED VARCHAR(50);
	DECLARE VALUE__ITEM_STATUS__ON_SALE VARCHAR(50);
	DECLARE VALUE__ITEM_STATUS__NOT_ON_SALE VARCHAR(50);

	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__INTEGRATED INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__PROD_CODE INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__PROD_NAME INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_NAME INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__BRAND_NAME INT;
	DECLARE VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_CODE INT;
    
    DECLARE VALUE__PERIOD_SEARCH_CRITERIA__PROD_REGIST_DATE INT;
    DECLARE VALUE__PERIOD_SEARCH_CRITERIA__SALE_START_DATE INT;
    DECLARE VALUE__PERIOD_SEARCH_CRITERIA__SALE_END_DATE INT;
    
    DECLARE ALONE_ITEMS_QUERY LONGTEXT;
    DECLARE BOUND_ITEMS_QUERY LONGTEXT;
    DECLARE S LONGTEXT;
    
    
	SET _isSearchingForItem = p_isSearchingForItem;
    SET _prodType = p_prodType;
    SET _displayedYn = p_displayedYn;
    SET _itemStatus = p_itemStatus;
    SET _partnerCompanyId = p_partnerCompanyId;
    SET _brandId = p_brandId;
    SET _isSoldOut = p_isSoldOut;
    SET _categoryId = p_categoryId;
    SET _keywordSearchKeyword = p_keywordSearchKeyword;
    SET _keywordSearchCriteria = p_keywordSearchCriteria;
    SET _periodSearchCriteria = p_periodSearchCriteria;
    SET _periodSearchRangeStart = p_periodSearchRangeStart;
    SET _periodSearchRangeEnd = p_periodSearchRangeEnd;
    SET _prodCode = p_prodCode;
    SET _itemCode = p_itemCode;
    SET _rowsPerPage = p_rowsPerPage;
    SET _startRowIndex = p_startRowIndex;

	SET VALUE__ITEM_STATUS__UNREGISTERED = 'UNREGISTERED';
	SET VALUE__ITEM_STATUS__ON_SALE = 'ON_SALE';
	SET VALUE__ITEM_STATUS__NOT_ON_SALE = 'NOT_ON_SALE';

	SET VALUE__KEYWORD_SEARCH_CRITERIA__INTEGRATED = 1;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__PROD_CODE = 2;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__PROD_NAME = 3;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_NAME = 4;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__BRAND_NAME = 5;
	SET VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_CODE = 6;
    
    SET VALUE__PERIOD_SEARCH_CRITERIA__PROD_REGIST_DATE = 1;
    SET VALUE__PERIOD_SEARCH_CRITERIA__SALE_START_DATE = 2;
    SET VALUE__PERIOD_SEARCH_CRITERIA__SALE_END_DATE = 3;


    -- 1. 묶이기 전 품목들 가져오는 쿼리 세팅하기(UNION하기 위함)
    SET ALONE_ITEMS_QUERY = '
		SELECT prod.prod_code, item.item_code, item.item_status, prod.prod_name, item.item_name, item.retail_price, item.purch_price, item.denall_price, item.monthly_goal_sale_cnt, item.annual_goal_sale_cnt, item.regist_datetime, item.approve_req_datetime, item.sale_start_date, emp.emp_name as manager_id, item.approve_confirm_datetime, item.sale_end_date, item.stock_cnt, item.item_kind, item.acc_sale_cnt, item.acc_sale_amount
		FROM item
		LEFT JOIN prod ON prod.prod_code = item.prod_code
		LEFT JOIN emp ON emp.emp_id = prod.manager_id
		WHERE prod.prod_code IS NULL ';
	IF (_prodType IS NOT NULL) THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.prod_type = ', _prodType, ' ');
	END IF;
	IF _displayedYn IS NOT NULL THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.displayed_yn = ''', _displayedYn, ''' ');
	END IF;
	IF _itemStatus IS NOT NULL THEN
		IF _itemStatus = VALUE__ITEM_STATUS__UNREGISTERED THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.item_status IN (1, 2, 3) ');
		END IF;
		IF _itemStatus = VALUE__ITEM_STATUS__ON_SALE THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.approve_confirm_datetime IS NOT NULL AND curdate() BETWEEN item.sale_start_date AND item.sale_end_date ');
		END IF;
		IF _itemStatus = VALUE__ITEM_STATUS__NOT_ON_SALE THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.approve_confirm_datetime IS NOT NULL AND curdate() > item.sale_end_date ');
		END IF;
	END IF;
    
	IF _partnerCompanyId IS NOT NULL THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.partner_company_id = ''', _partnerCompanyId, ''' ');
	END IF;

	IF _brandId IS NOT NULL THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.brand_id = ''', _brandId, ''' ');
	END IF;
	
	IF _isSoldOut IS NOT NULL THEN
		IF _isSoldOut = true THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.stock_cnt <= 0 is true ');
		END IF;
		IF _isSoldOut = false THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND item.stock_cnt <= 0 is false ');
		END IF;
	END IF;

	IF _categoryId IS NOT NULL THEN
		SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.category_id = ''', _categoryId, ''' ');
	END IF;

	IF _keywordSearchKeyword IS NOT NULL THEN
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__INTEGRATED THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND ( MATCH (item.item_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_code) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_short_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH (prod.prod_desc) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR prod.brand_id = (SELECT brand_id FROM brand WHERE MATCH(brand_name) AGAINST (''', _keywordSearchKeyword, ''')) ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH item.item_code AGAINST (''', _keywordSearchKeyword, ''') ');
				SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'OR MATCH item.item_name AGAINST (''', _keywordSearchKeyword, ''') ');
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, ') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__PROD_CODE THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND MATCH prod.prod_code AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__PROD_NAME THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND MATCH prod.prod_name AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__BRAND_NAME THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND prod.brand_id = (SELECT brand_id FROM brand WHERE MATCH(brand_name) AGAINST (''', _keywordSearchKeyword, ''')) ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_NAME THEN
			SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'AND MATCH item.item_name AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
	END IF;
    SET ALONE_ITEMS_QUERY = CONCAT(ALONE_ITEMS_QUERY, 'ORDER BY item.approve_req_datetime ');	


	-- 2. 묶인 후 품목들 가져오는 쿼리 세팅하기--
    SET BOUND_ITEMS_QUERY = '
		SELECT prod.prod_code, item.item_code, item.item_status, prod.prod_name, item.item_name, item.retail_price, item.purch_price, item.denall_price, item.monthly_goal_sale_cnt, item.annual_goal_sale_cnt, item.regist_datetime, item.approve_req_datetime, item.sale_start_date, emp.emp_id as manager_id, item.approve_confirm_datetime, item.sale_end_date, item.stock_cnt, item.item_kind, item.acc_sale_cnt, item.acc_sale_amount
		FROM item
		LEFT JOIN prod ON prod.prod_code = item.prod_code
		LEFT JOIN emp ON emp.emp_id = prod.manager_id
		WHERE prod.prod_code IS NOT NULL ';

	IF (_prodType IS NOT NULL) THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.prod_type = ', _prodType, ' ');
	END IF;
	IF _displayedYn IS NOT NULL THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.displayed_yn = ''', _displayedYn, ''' ');
	END IF;
	IF _itemStatus IS NOT NULL THEN
		IF _itemStatus = VALUE__ITEM_STATUS__UNREGISTERED THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.item_status IN (1, 2, 3) ');
		END IF;
		IF _itemStatus = VALUE__ITEM_STATUS__ON_SALE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.approve_confirm_datetime IS NOT NULL AND curdate() BETWEEN item.sale_start_date AND item.sale_end_date ');
		END IF;
		IF _itemStatus = VALUE__ITEM_STATUS__NOT_ON_SALE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.approve_confirm_datetime IS NOT NULL AND curdate() > item.sale_end_date ');
		END IF;
	END IF;
    
	IF _partnerCompanyId IS NOT NULL THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.partner_company_id = ''', _partnerCompanyId, ''' ');
	END IF;

	IF _brandId IS NOT NULL THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.brand_id = ''', _brandId, ''' ');
	END IF;
	
	IF _isSoldOut IS NOT NULL THEN
		IF _isSoldOut = true THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.stock_cnt <= 0 is true ');
		END IF;
		IF _isSoldOut = false THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.stock_cnt <= 0 is false ');
		END IF;
	END IF;

	IF _categoryId IS NOT NULL THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.category_id = ''', _categoryId, ''' ');
	END IF;

	IF _keywordSearchKeyword IS NOT NULL THEN
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__INTEGRATED THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND ( MATCH (item.item_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_code) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_short_name) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH (prod.prod_desc) AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR prod.brand_id = (SELECT brand_id FROM brand WHERE MATCH(brand_name) AGAINST (''', _keywordSearchKeyword, ''')) ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH item.item_code AGAINST (''', _keywordSearchKeyword, ''') ');
				SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'OR MATCH item.item_name AGAINST (''', _keywordSearchKeyword, ''') ');
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, ') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__PROD_CODE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND MATCH prod.prod_code AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__PROD_NAME THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND MATCH prod.prod_name AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__BRAND_NAME THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.brand_id = (SELECT brand_id FROM brand WHERE MATCH(brand_name) AGAINST (''', _keywordSearchKeyword, ''')) ');
		END IF;
		IF _keywordSearchCriteria = VALUE__KEYWORD_SEARCH_CRITERIA__ITEM_NAME THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND MATCH item.item_name AGAINST (''', _keywordSearchKeyword, ''') ');
		END IF;
	END IF;
    
	IF _periodSearchCriteria IS NOT NULL THEN
		IF _periodSearchCriteria = VALUE__PERIOD_SEARCH_CRITERIA__PROD_REGIST_DATE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.approve_confirm_datetime BETWEEN ''', _periodSearchRangeStart, ''' AND ''', _periodSearchRangeEnd, ''' ');
		END IF;
		IF _periodSearchCriteria = VALUE__PERIOD_SEARCH_CRITERIA__SALE_START_DATE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.sale_start_date BETWEEN ''', _periodSearchRangeStart, ''' AND ''', _periodSearchRangeEnd, ''' ');
		END IF;
		IF _periodSearchCriteria = VALUE__PERIOD_SEARCH_CRITERIA__SALE_END_DATE THEN
			SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND item.sale_end_date BETWEEN ''', _periodSearchRangeStart, ''' AND ''', _periodSearchRangeEnd, ''' ');
		END IF;
	END IF;

	IF _isSearchingForItem = true THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'AND prod.prod_code = ''', _prodCode, ''' AND item.item_code != ''', _itemCode, ''' ');
	END IF;
	IF _isSearchingForItem = false THEN
		SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'GROUP BY prod.prod_code ');
	END IF;

    SET BOUND_ITEMS_QUERY = CONCAT(BOUND_ITEMS_QUERY, 'ORDER BY item.approve_req_datetime ');
    
    -- ----------------
    SET S = 'SELECT * FROM (';
	SET S = CONCAT(S, '(', BOUND_ITEMS_QUERY,')'); -- 묶인 후 쿼리
	IF (_isSearchingForItem = false AND _periodSearchCriteria IS NULL) THEN -- 사실 묶이기 전 쿼리는 이 조건일때만 UNION해야 함
		SET S = CONCAT(S, ' UNION (', ALONE_ITEMS_QUERY, ')');
	END IF;
    
    SET S = CONCAT(S, ') X LIMIT ', _rowsPerPage, ' OFFSET ', _startRowIndex, ';');
    -- ----------------
	SET @S = S;
    PREPARE stmt FROM @S;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END $$
DELIMITER ;

-- 아래는 테스트용도로 호출하는 쿼리
-- CALL getItems(
-- 	false --     _isSearchingForItem BOOLEAN,
-- 	,NULL --     _prodType INT,
-- 	,NULL --     _displayedYn CHAR(1),
-- 	,NULL --     _itemStatus INT,
-- 	,NULL --     _partnerCompanyId VARCHAR(50),
-- 	,NULL --     _brandId VARCHAR(50),
-- 	,NULL --     _isSoldOut BOOLEAN,
-- 	,NULL --     _categoryId VARCHAR(50),
-- 	,NULL --     _keywordSearchKeyword VARCHAR(200),
-- 	,NULL --     _keywordSearchCriteria INT,
-- 	,NULL --     _periodSearchCriteria INT,
-- 	,NULL --     _periodSearchRangeStart VARCHAR(20),
-- 	,NULL --     _periodSearchRangeEnd VARCHAR(20),
-- 	,NULL --     _prodCode VARCHAR(50),
-- 	,NULL --     _itemCode VARCHAR(50),
-- 	,10 -- 		 _rowsPerPage INT,
-- 	,0 -- 		 _startRowIndex INT
-- );

```


그래서 작성한 쿼리문은 다음과 같다.
```xml
<mapper namespace="com.mycompany.backend.dao.ProductDao">

	<!-- 필터검색 개수 세는 프로시저 -->
	<select id="count" parameterType="filter" resultType="int" statementType="CALLABLE">
		{ CALL 
			countItems(
				#{prodType,				mode=IN, jdbcType=INTEGER},
				#{displayedYn,			mode=IN, jdbcType=CHAR},
				#{itemStatus,			mode=IN, jdbcType=VARCHAR},
				#{partnerCompanyId,		mode=IN, jdbcType=VARCHAR},
				#{brandId,				mode=IN, jdbcType=VARCHAR},
				#{isSoldOut,				mode=IN, jdbcType=BOOLEAN},
				#{categoryId,			mode=IN, jdbcType=VARCHAR},
				#{keywordSearchKeyword,	mode=IN, jdbcType=VARCHAR},
				#{keywordSearchCriteria,	mode=IN, jdbcType=INTEGER},
				#{periodSearchCriteria,	mode=IN, jdbcType=INTEGER},
				#{periodSearchRangeStart,mode=IN, jdbcType=VARCHAR},
				#{periodSearchRangeEnd,	mode=IN, jdbcType=VARCHAR}
			)
		}
	</select>

	<!-- 필터검색 검색하는 프로시저 -->
	<select id="selectByFilterAndPage" resultType="item" statementType="CALLABLE">
		{ CALL 
			getItems(
				#{isSearchingForItem,			mode=IN, jdbcType=BOOLEAN},
				#{filter.prodType,				mode=IN, jdbcType=INTEGER},
				#{filter.displayedYn,			mode=IN, jdbcType=CHAR},
				#{filter.itemStatus,			mode=IN, jdbcType=VARCHAR},
				#{filter.partnerCompanyId,		mode=IN, jdbcType=VARCHAR},
				#{filter.brandId,				mode=IN, jdbcType=VARCHAR},
				#{filter.isSoldOut,				mode=IN, jdbcType=BOOLEAN},
				#{filter.categoryId,			mode=IN, jdbcType=VARCHAR},
				#{filter.keywordSearchKeyword,	mode=IN, jdbcType=VARCHAR},
				#{filter.keywordSearchCriteria,	mode=IN, jdbcType=INTEGER},
				#{filter.periodSearchCriteria,	mode=IN, jdbcType=INTEGER},
				#{filter.periodSearchRangeStart,mode=IN, jdbcType=VARCHAR},
				#{filter.periodSearchRangeEnd,	mode=IN, jdbcType=VARCHAR},
				#{prodCode,						mode=IN, jdbcType=VARCHAR},
				#{itemCode,						mode=IN, jdbcType=VARCHAR},
				#{pager.rowsPerPage,			mode=IN, jdbcType=INTEGER},
				#{pager.startRowIndex,			mode=IN, jdbcType=INTEGER}
			)
		}
	</select>
  (이하 생략)
```

### 참고
* [Spring MyBatis 에서 Procedure (프로시저) 사용하기](https://blog.naver.com/jhj9512z/222244283964)
