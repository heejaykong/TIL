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

아무튼 위와 같이 투두리스트를 작성하고, 일을 시작했다. 이제 그 과정을 아래 기록하고자 한다.

## 1. 페이지 로딩시간 찍어내기

우선 현재의 문제를 정확히 파악하기 위해, 그리고 만약 로딩속도가 개선이 됐을 경우 얼마나 개선이 됐는지 보고하기 위해,

이 페이지가 로딩될 때 얼마나 시간을 잡아먹는지 기록해주는 코드를 작성하는 것이 필요해 보였다.

따라서 구글링을 해보니 장고에선 middleware를 구현하는 방법이 있었다.

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
```python
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
    def the_method(self):
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

여러모로 리팩토링이나 구조적인 개선이 있으면 좋겠지만, 우선은 쿼리 코드 최적화에만 집중해보기로 했다.

이어서 작성할 예정...
