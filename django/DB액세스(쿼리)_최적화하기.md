**2021. 12. 26. 기록**

# django DB액세스(쿼리) 최적화하기
### 부제: 인턴 중 목록페이지 속도 개선 과제를 진행하며 배운 것들

인턴 과제로서, 어떤 페이지에서 어떤 목록을 로딩할 때 시간이 너무 오래 걸리는 현상을 해결해야 했다.

기존의 로딩 속도는 (물론 목록이 갖고 있는 데이터 양에 따라 달랐지만) 보통 5초 이상은 소요하는 것 같았다.

느린 화면을 개선하라는 과제가 주어지니 막상 뭐부터 해야 하는지 몰라 막막하기만 했다.

그래서 단계별로 해야 할 일을 나름대로 미리 정리해보았다.

1. 페이지 로딩시간 찍어내기
2. 문제의 페이지를 로딩할 때 쓰이는 기존의 코드를 읽어보기
3. 로딩속도를 개선하는 보통의 방법이 있는지 리서치하기(장고 위주로)
4. 기존 코드의 연산낭비 여부를 마침내 찾아내고 최적화하기

이렇게 할일을 미리 정리하니 머릿속이 복잡하지도 않았고, 일을 차근차근 진행할 수 있었다.

(사수님이 항상 깃헙 이슈에 댓글로 과제를 위한 투두리스트를 먼저 공유하고 일을 시작하라고 한 이유가 아마 이것 때문이지 않을까 싶다.)

아무튼 위와 같이 투두리스트를 작성한 뒤 일을 시작했다. 이제 그 과정을 아래 기록하고자 한다.

## 1. 페이지 로딩시간 찍어내기

우선 현재의 문제를 정확히 파악하기 위해, 그리고 만약 로딩속도가 개선이 됐을 경우 얼마나 개선이 됐는지 보고하기 위해,

이 페이지가 로딩될 때 얼마나 시간을 잡아먹는지 기록해주는 코드를 작성하는 것이 필요해 보였다.

구글링을 해보니 장고에선 middleware를 구현하는 방법이 있었다.

관련 자료는 [여기](https://github.com/heejaykong/TIL/blob/main/django/%ED%8E%98%EC%9D%B4%EC%A7%80_%EB%A1%9C%EB%94%A9%EC%8B%9C%EA%B0%84_%EA%B8%B0%EB%A1%9D%ED%95%98%EB%8A%94_middleware_%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0.md)에 작성했다.

<img alt="로딩속도" src="https://user-images.githubusercontent.com/18097984/147402549-f12503cb-aba0-4f0d-9272-08be4b27f21e.png" />

구현한 미들웨어로 페이지 로딩시간을 찍어본 결과, 로컬 기준으로는 6 ~ 7초 정도가 걸리는 것으로 파악됐다.(`duration`)

쿼리 호출횟수(`db_queries`)는 128회 ~ 129회 정도였고 DB에 액세스하는 시간(`db_time`)이 소요시간의 거의 대부분을 차지했다.

즉 해당 페이지를 로드하기 위해 실행되는 DB 액세스 코드를 최적화하면 로딩속도가 개선되지 않을까? 하는 생각을 할 수 있었다.

그리고 문제의 화면을 로드할 때 미들웨어에서 `print(connection.queries)`를 하면, 호출된 129여개의 `db_queries`가 정확히 어떤 쿼리들인지 그 내역을 목록으로 확인할 수 있는데,

특정 데이터만 유독 호출이 반복되는 것처럼 보이는 쿼리들이 많았다.

<img alt="쿼리내역목록" src="https://user-images.githubusercontent.com/18097984/147402914-65774d89-103c-432c-86b8-cc23db78b2fd.png">

_쿼리내역 일부 모습(내용은 지웠다...)_

## 2. 문제의 페이지를 로딩할 때 쓰이는 기존의 코드를 읽어보기

이제 시간을 잡아먹는 요인이 정확히 어떤 건지 기존 코드를 파악해보며 알아낼 차례다.

_some_template.html_
```HTML+Django
...
<table>
    ...
    <tbody>
        {% for project in projects %}
            <tr>
                ...
                <td>{{ project.the_method }}</td>
                ...
            </tr>
        {% endfor %}
    </tbody>
</table>
...
```

_views.py_
```python
def some_view_function(request):
    ...
    projects = TheModel.objects.all().order_by("`...`", "`...`")
    ...
    return render(
        request,
        ".../some_template.html",
        {
            "projects": queryset,
            ...
        }
    )
```

_models.py_
```python
...
class TheModel(models.Model):
    ...
    (필드 및 다른 메소드 정의)
    ...
    def the_method(self): # 유독 많이 호출되는 것으로 보였던 쿼리의 장본인 메소드
        # `TheModel`에 속하는 다른 특정 모델 쿼리셋의 length들을 스트링으로 리턴하는 메소드.
        dummy = SomeOtherModel.objects.filter(`...`=self.`...`)
        dummy2 = dummy.values_list('some_id', flat=True)
        if not dummy.first():
            return
        else:
            some_int = dummy.first().some_int_field
            done = AnotherModel.objects.filter(`...`=self.`...`, some_condition=False)
            return str(len(done)) + '/' + str(len(dummy2) * some_int) # "숫자/숫자" 형식의 스트링을 리턴
...
```

대충 유출이 되지 않는 선에서 적어보자면 위와 비슷한 코드였다.

* `the_method`: `TheModel`에서 특정 쿼리셋의 length들을 스트링으로 리턴하는 메소드.
* `dummy`: 어떤 조건에 맞는 `SomeOtherModel`들의 쿼리셋.
* `dummy2`: 어떤 조건에 맞는 `SomeOtherModel`들의 `some_id`만 가져온 쿼리셋.
* `some_int`: `SomeOtherModel` 쿼리셋 중 하나의(어떤 것이든 상관이 없는지 여기선 `first()`의 것을 계산하고 있다) `some_int_field`값.
* `done`: 어떤 조건에 맞는 `AnotherModel`들의 쿼리셋.
* 리턴문에서는 done의 길이, 그리고 dummy2의 길이와 some_int를 계산한 값을 스트링으로 리턴하고 있다.

(여러모로 리팩토링이나 구조적인 개선이 있으면 좋겠지만, 우선은 쿼리 코드 최적화에만 집중하기로 했다.)

코드를 대강 훑어봤으니 이제 다음 단계로 넘어가자.

## 3. 로딩속도를 개선하는 보통의 방법이 있는지 리서치하기(장고 위주로)

앞서 파악한 결과 쿼리 코드들을 최적화하면 문제가 개선될 것이라고 생각해, 장고 쿼리 최적화 위주로 검색해보았더니 다음과 같은 자료들을 찾을 수 있었다.

* [[django 공식문서] Database access optimization](https://docs.djangoproject.com/ko/2.1/topics/db/optimization/)
* [Django에서 DB 액세스 최적화하기](https://blog.myungseokang.dev/posts/database-access-optimization/)
* [Django queryset filter와 exists()](https://daeguowl.tistory.com/66?category=823041)

마침 나의 케이스에 딱 맞는 방법들이 소개돼 있었다.

정리해보자면 다음과 같다.

> QuerySet 계산 이해하기

>> QuerySet은 Lazy하다: QuerySet을 생성하는 행위는 데이터베이스 활동을 포함하지 않습니다. Django는 QuerySet이 계산될 때까지 실제로 쿼리를 실행하지 않습니다. 실제로 계산되는 시점은 아래에서 더 자세하게 다룹니다.

>> QuerySet은 다음과 같은 시점에 계산됩니다: 반복 (Iteration), 슬라이싱 (Slicing), 피클링 / 캐싱 (Pickling/Caching), repr(), len(), list(), bool()

>> 데이터가 메모리에 저장되는 방식: 각 QuerySet에는 데이터베이스 액세스를 최소화하기 위한 캐시가 포함되어 있습니다. 이것이 어떻게 작동하는지 이해하면 효율적인 코드를 작성하는 데 도움이 됩니다.

> 필요없는 항목은 검색하지 마세요

>> QuerySet.values() 및 values_list()를 사용하기: dict 또는 list 값을 원할 때, ORM 모델 객체가 필요하지 않은 경우에는 values()를 적절하게 사용하세요. 템플릿 코드에서 모델 객체를 대체하는 데 유용할 수 있습니다.

>> QuerySet.count() 사용하기: 개수만 원하는 경우에 len(queryset)보다 적절합니다.

>> QuerySet.exists() 사용하기: 적어도 하나의 결과가 존재하는지 알아내고 싶은 경우에 if queryset보다 적절합니다.

>> (단, count() 및 exists()를 과도하게 사용하지 말고 적절히 상황따라 판단해야 한다.)


## 4. 기존 코드의 연산낭비 여부를 마침내 찾아내고 최적화하기

3번 단계에서 쌓은 지식을 갖고, 다시 models.py 코드로 돌아가 문제점을 차례대로 파악해 보자.

_models.py_
```python
...
class TheModel(models.Model):
    ...
    def the_method(self):
        dummy = SomeOtherModel.objects.filter(`...`=self.`...`)  # ...(1)
        dummy2 = dummy.values_list('some_id', flat=True)  # ...(2)
        if not dummy.first():  # ...(3)
            return
        else:
            some_int = dummy.first().some_int_field  # ...(4)
            done = AnotherModel.objects.filter(`...`=self.`...`, some_condition=False)
            return str(len(done)) + '/' + str(len(dummy2) * some_int)  # ...(5)
...
```

* (1) ~ (2)번째 라인:
    * 어떤 조건에 맞는 `SomeOtherModel`들의 쿼리셋을 `dummy`에 담고, `dummy`의 `some_id`를 `dummy2`에 또 담는 식이다.
    * 이 메소드에서 필요한 건 오로지 `SomeOtherModel`의 `some_id`이다. 따라서 (1)에서 `dummy`에 `SomeOtherModel`를 filter한 객체들의 모든 항목을 가져오는 것은 연산 낭비로 보인다.
    * **처음부터 `values_list()`만을 사용해 필요한 항목만 검색하는 방식으로 최적화를 시도할 수 있다.**
* (3)번째 라인:
    * 이 코드를 작성한 사람은 `first()`로써 오로지 쿼리셋의 존재 여부만을 판별하고 싶었던 것으로 보인다.
    * **쿼리셋의 존재 여부를 판별하기 위해서는 `first()` 대신 `exists()`를 사용할 수 있다.**
* (4)번째 라인:
    * 모든 항목을 가져온 `dummy`에서 첫 번째 객체의 `some_int_field`를 `some_int`에 담는 식이다.
    * 이 역시 앞서 말한 것처럼 `values_list()`로 `some_id`만 가져온 쿼리셋만으로도 첫 번째 객체의 `some_int_field`값을 구할 수 있을 것이다.
    * **`get()`을 사용하면 가능할 것으로 보인다.**
* (5)번째 라인:
    * 오로지 개수를 세기 위해 `len(done)`과 `len(dummy2)`가 쓰이는데, 연산 낭비로 보인다.
    * **`count()`라는 더 좋은 함수로 대체하면 될 것으로 보인다.**

### 따라서 아래와 같이 코드를 고쳐보았다.

_models.py 수정된 버전_
```diff
...
class TheModel(models.Model):
    ...
    def the_method(self):
-       dummy = SomeOtherModel.objects.filter(`...`=self.`...`)
-       dummy2 = dummy.values_list('some_id', flat=True)
-       if not dummy.first():
+       new_dummy = SomeOtherModel.objects.filter(`...`=self.`...`).values_list('some_id', flat=True)
+       if not new_dummy.exists():
            return
        else:
-           some_int = dummy.first().some_int_field
+           some_int = SomeOtherModel.objects.get(some_id=new_dummy.first()).some_int_field
            done = AnotherModel.objects.filter(`...`=self.`...`, some_condition=False)
-           return str(len(done)) + '/' + str(len(dummy2) * some_int)
+           return str(done.count()) + '/' + str(new_dummy.count() * some_int)
...
```

위처럼 `values_list()`, `exists()`, `get()`, `count()`를 적절히 사용하니 로딩속도가 훨씬 개선됐다.

공식문서에서 하라는 대로 하고 실제로 개선이 되는 순간을 목격하니까 정말 신기했다.

<img alt="로딩속도" src="https://user-images.githubusercontent.com/18097984/147402549-f12503cb-aba0-4f0d-9272-08be4b27f21e.png" />

<img alt="개선된로딩속도" src="https://user-images.githubusercontent.com/18097984/147432889-3c40cb2c-c4f8-436a-8f8e-038f5b91e822.png">

_개선된 로딩속도. 이전에 6 ~ 7초가 소요되던 것과는 달리 지금은 1 ~ 2초 정도만 소요된다._

+) 수정된 코드를 이용하는 또 다른 기능에서도 덩달아 개선효과가 나타났다.

<img alt="기존로딩속도2" src="https://user-images.githubusercontent.com/18097984/151532963-fdabb403-9349-4823-a8f7-f013104aabbd.png">

<img alt="개선된로딩속도2" src="https://user-images.githubusercontent.com/18097984/151532937-d47b9f66-3525-4f89-acb5-ade3d1ab699c.png">

_18 ~ 20초가 소요되던 화면이 5초 정도만 소요된다._

이번 과제로 django만의 QuerySet의 특징, QuerySet 캐시 활용, 효율적인 DB 접근 방식 등을 익힐 수 있어 좋았을 뿐더러, 실제 프로덕트의 사용성 개선에도 기여할 수 있어 뿌듯했다.

### 참고
* [20 Ways to Speed Up Your Website and Improve Conversion in 2022](https://www.crazyegg.com/blog/speed-up-your-website/)
* [Django: display time it took to load a page on every page](https://stackoverflow.com/questions/17751163/django-display-time-it-took-to-load-a-page-on-every-page)
* [How to calculate response time in Django](https://pretagteam.com/question/how-to-calculate-response-time-in-django)
* [Show page generation time in Django - Stavros' Stuff](https://www.stavros.io/posts/django-page-load-time/)
* [[django 공식문서] Database access optimization](https://docs.djangoproject.com/ko/2.1/topics/db/optimization/)
* [Django에서 DB 액세스 최적화하기](https://blog.myungseokang.dev/posts/database-access-optimization/)
* [Django queryset filter와 exists()](https://daeguowl.tistory.com/66?category=823041)
