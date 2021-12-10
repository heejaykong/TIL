# 페이지 로딩시간 기록하는 middleware 구현하기

**2021.12.10. 기록**

pagination으로 페이징을 할때마다 속도가 너무 느려지는 화면이 있었다.

원인을 찾고 속도개선을 하기 전에 우선 페이지를 로딩하는 시간을 수치화해서보면 좋겠다는 생각이 들었다.

calculate time when page load django <- 대충 이런 키워드로 구글링을 해보니 middleware를 구현하는 방법이 있었다.

여러가지 글들을 참고해 짬뽕한 내 middleware 코드는 다음과 같다. (function이나 class 다 가능해보인다. 나는 class로 구현했다)

```python
# stats_middleware.py(settings.py와 같은 경로에 위치하게 만듦)
from django.db import connection
from functools import reduce
from operator import add
import time


class StatsMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        
        # get number of db queries before we do anything
        n = len(connection.queries)
        # print("n!!!!!!!!!!!!")
        # print(n)

        # time the view
        start_time = time.time()
        response = self.get_response(request)
        duration = time.time() - start_time

        # compute the db time for the queries just run
        db_queries = len(connection.queries) - n
        # print("db_queries!!!!!!!!!!!!!!!!!!!!")
        # print(connection.queries)
        # print(db_queries)
        
        if db_queries:
            db_time = reduce(add, [float(q['time']) for q in connection.queries[n:]])
        else:
            db_time = 0.0

        # and backout python time
        python_time = duration - db_time

        stats = {
            'duration': duration,
            'python_time': python_time,
            'db_time': db_time,
            # 'db_queries': db_queries,
        }

        # Add the header.Or do other things, my use case is to send a monitoring metric
        # print("")
        # print("")
        # print("stats!!!!!!!!!!!!!!!!!!!!")
        # print(stats)

        return response
```

```python
# settings.py를 다음과 같이 수정
...
MIDDLEWARE = [
    'mysite.stats_middleware.StatsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    ...
]
...
```

참고로 이 글을 작성할 때 사용한 Django 버전은 2.1.7이다.

### 참고
* [Django: display time it took to load a page on every page](https://stackoverflow.com/questions/17751163/django-display-time-it-took-to-load-a-page-on-every-page)
* [How to calculate response time in Django](https://pretagteam.com/question/how-to-calculate-response-time-in-django)
* [Show page generation time in Django - Stavros' Stuff](https://www.stavros.io/posts/django-page-load-time/)
