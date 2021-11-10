**2021.11.10 기록**

# 장고로 pagination 토막내는 방법

장고로 개발된, 기존에 있는 페이지네이터를 고쳐야 했다.

기존의 페이지네이터는 만약 페이지가 1부터 55까지 있다면 그 모든 페이지 버튼이 다 보이는 바람에 불편을 초래했다.

그래서 5개씩이든 10개씩이든 페이지 버튼이 보여지는 범위가 한정되게끔 개선하는 과제가 주어졌다 ㅎㅎ

장고로 개발해보는 것도 처음이라 첨엔 막막했는데 찾다보니 장고 3.2. 버전부터는 `Paginator.get_elided_page_range`라는 메소드를 쓸 수 있는 것으로 보였다.

그리하여 다음과 같은 형태로, 범위를 벗어나는 페이지 버튼은 ellipsis로 대체되도록 개선할 수 있었다.

![image](https://user-images.githubusercontent.com/18097984/141092736-1020e401-893e-4c8e-b205-4390f4770a4e.png)
![image](https://user-images.githubusercontent.com/18097984/141092770-56d79709-18a6-46f0-83b5-c5a5b5955bca.png)
![image](https://user-images.githubusercontent.com/18097984/141092787-7b7dabdc-4dae-4139-bf76-5da8a9a19e7a.png)

자세한 내용은 아래 참고링크를 차례대로 확인하자.

### 참고
* [Display only some of the page numbers by django pagination](https://stackoverflow.com/a/66772817)
* [django doc - Paginator - methods - Paginator.get_elided_page_range()](https://docs.djangoproject.com/en/3.2/ref/paginator/#django.core.paginator.Paginator.get_elided_page_range)
* [How to use elided pagination in Django and solve too many pages problem](https://nemecek.be/blog/105/how-to-use-elided-pagination-in-django-and-solve-too-many-pages-problem)
