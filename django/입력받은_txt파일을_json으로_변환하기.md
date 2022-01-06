**2021.01.06. 기록**
# 입력받은 txt파일을 json으로 변환하기

ModelForm에서 FileField로 .txt 파일을 입력 받고, 그 파일 내부에 각 줄마다 적힌 json 데이터를 일일이 post해야 했다.

(참고로 무조건 똑같은 json 포맷의 string이 적힌 .txt파일을 받는다는 게 보장되어 있는 구조였다)

입력되는 .txt 파일은 늘 다음과 같은 형태였다.

_data.txt_
```txt
{'A': {'B': 'http://some/random/url1.jpeg'}}
{'A': {'B': 'http://some/random/url2.jpeg'}}
{'A': {'B': 'http://some/random/url3.jpeg'}}
...
```

json.loads()로 변환하려다보니 `JSONDecodeError`가 뜨더라. 에러메시지는 `Extra data: line 2 column 1 (char 116)` 이런 식이었다.

검색해보니 각 json 객체가 쉽표 없이 나열되기만 하는 구조라, 즉 여러개의 json 객체가 존재하는 거라 json.loads()로 파일 내용 전체를 한번에 변환할 수는 없어서 발생하는 오류였다.

따라서 아래와 같이 작성했다
```python
...
if form.is_valid():
  txt_file = request.FILES['the_filefield']
  data_list = [json.loads(line) for line in txt_file.readlines()] # <--- 요 부분
  for data in data_list:
    # do the post
...
```

이번 삽질로 `read()`, `readlines()`, `json.loads()` 등을 배울 수 있었다.

### 참고
* [Parsing multiple json objects from a text file using Python](https://stackoverflow.com/questions/51752925/parsing-multiple-json-objects-from-a-text-file-using-python)
