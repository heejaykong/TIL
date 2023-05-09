# 집계함수_count()_유의점.md

**2023. 05. 08. 기록**

데이터베이스 복습하다 새삼 깨달은 것 기록하기

**SQL에서 집계함수(aggregate function)는 null값은 제외하고 계산을 한다**

따라서 count()같은 집계함수 사용 시 만약 null값이 포함된 칼럼을 갖다가 count()하게 되면 원하는 결과가 나오지 않을 수 있기에

보통 count(PK 속성) 또는 count(\*)를 한다

![count()유의1](https://user-images.githubusercontent.com/18097984/236709347-ed8ba801-bcbd-4959-8b0f-80c3701f10d2.jpg)
![count()유의2](https://user-images.githubusercontent.com/18097984/236709351-260ea9ed-9465-45ab-a221-344cbce5099d.jpg)

### 참고
* [kocw - 데이터베이스의 원리와 응용(한양대학교 백현미)](http://www.kocw.net/home/search/kemView.do?kemId=1163794)
