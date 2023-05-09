# group_by_유의점.md

**2023. 05. 09. 기록**

Group by가 쓰인 쿼리문의 Select절에서는, 집계함수나 group by 절에 명시된 속성이 아니면 쓰지 말자.

![GroupBy유의점](https://user-images.githubusercontent.com/18097984/236961922-7e2ae9d8-9fcc-4d87-b3f1-f47db17dac32.jpg)

이유? Group by `주문고객`을 하면 그 주문고객 을 기준으로 데이터들이 묶여야 하는데,

집계함수로 감싸지 않은 뜬금없는 `배송지`를 select 해버리면 뭘 기준으로 `배송지`의 단일값을 가져와야 할지 알 수 없어져 버리기 때문.

### 참고
* [kocw - 데이터베이스의 원리와 응용(한양대학교 백현미)](http://www.kocw.net/home/search/kemView.do?kemId=1163794)
