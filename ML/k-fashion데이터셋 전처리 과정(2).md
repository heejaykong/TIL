**2021. 09. 05. 기록**

# K-Fashion 이미지 데이터셋을 전처리해 보자 (2) - Data Cleaning 편

## 1. 배경설명

> **이전 포스트: [K-Fashion 이미지 데이터셋을 전처리해 보자 (1) - Data Reduction 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(1).md#k-fashion-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%EC%9D%84-%EC%A0%84%EC%B2%98%EB%A6%AC%ED%95%B4-%EB%B3%B4%EC%9E%90-1---data-reduction-%ED%8E%B8)**

이전 포스트에서 설명했듯이 [K-Fashion 이미지 데이터셋](https://aihub.or.kr/aidata/7988)을 활용해,

사용자가 옷차림 이미지를 입력하면 이미지에 나타난 상의의 카테고리와 소매기장을 반환해주는

**Multi-label image classification** 모델을 만들어야 했다.

데이터 전처리 과정은 아래처럼 4단계로 나눴다.

1. Data Reduction: 전체 데이터 중 상의 모델학습에 필요한 정보만 뽑아 하나의 csv 파일로 만들기
2. Data Cleaning: 오염된 데이터들 걸러내기
3. Generate Train, Validation, Test: 원하는 양만큼 train, valid, test 데이터 각각 생성하기
4. Move Image: 데이터셋에 해당하는 이미지들 원하는 파일로 옮기기

이번 편에서는 **2. Data Cleaning** 단계를 다뤄보겠다.

이번 포스트도 마찬가지로 본문의 코드 예시는 train 데이터에 해당되는 코드만 있다. **전체코드는 마지막에 첨부**했다.

## 2. 본문

이전 포스트에서는 전처리 당할 준비가 된 `raw_train`과 `raw_valid` 데이터셋을 각각 만들었다.

<img src="https://user-images.githubusercontent.com/18097984/136834983-625c3f8c-e433-431c-95f2-da34c2b34e0c.png" width="25%" alt="screenshot" />

*생성된 각각의 데이터셋 파일*

두 파일을 면밀히 살펴본 결과 아래와 같이 3개의 과정이 필요해 보였다.
1) 파일명 확장자 통일하기
2) 중복 및 모호한 데이터 걸러내기
3) train과 valid 사이 중복 데이터 또 걸러내기

그러면 차례대로 살펴보자.

### 1) 파일명 확장자 통일하기

![image](https://user-images.githubusercontent.com/18097984/136968943-3517f044-ab6b-4ca5-a03c-ff2fd6bc4628.png)

시행착오 끝에 알아낸 사실이 있다. 이 `id`와 `file_name` 값들이 unique하지 않다는 것이다.

어차피 나중에 `file_name`으로 이미지데이터를 옮기는 작업도 진행할 예정이니까 좀 더 편리해보이는 `file_name`을 기준으로 전처리를 진행했다.

**근데 자세히 보니 파일명 확장자들이 통일되어있지 않았다.**

<img src="https://user-images.githubusercontent.com/18097984/136966095-7e44fbbc-c29f-4e2d-befb-66a1d729d657.png" width="50%" alt="screenshot" />

*확장자 `.jpg`, `.JPG`가 섞인 모습*

**왜 이게 문제가 되는가?**

`file_name`값이 전부 서로 unique하다는 게 보장된다면 확장자를 통일시킬 필요가 없겠지만, 파일명이 겹치는 경우가 있다는 것을 알고 있기에

(파일명이 동일해도 확장자가 다를 경우 다른 string으로 인식되니까) 전처리 편의성을 위해, 확장자들을 `.jpg`로 통일시키기로 했다.

먼저, 기존의 `raw_train` 파일을 읽어온 뒤, `replace_extensions()` 함수에 넣었다.

```python
# reading the csv file
base = "../../input/"
RAW_TRAIN_PATH = os.path.join(base, "raw_train.csv")
raw_train = pd.read_csv(RAW_TRAIN_PATH, encoding="utf-8")
print(len(raw_train))
```
<img src="https://user-images.githubusercontent.com/18097984/136959450-d6d7b370-b2e7-454b-97b4-e93a75c27809.png" width="45%" alt="screenshot" />

*실행결과 `raw_train`의 데이터 수는 총 415,707개다*

```python
raw_train = replace_extensions(raw_train)
```

처리된 결과를 보자. 왼쪽이 처리되기 전 모습, 오른쪽이 `replace_extensions()`를 거친 모습이다.

<div style="display:flex;">
  <img src="https://user-images.githubusercontent.com/18097984/136966095-7e44fbbc-c29f-4e2d-befb-66a1d729d657.png" width="45%" alt="screenshot" />
  =>
  <img src="https://user-images.githubusercontent.com/18097984/136966505-35f82787-2f47-40b7-8205-11da5cdabfe9.png" width="45%" alt="screenshot" />
</div>

그렇다면 이제 `replace_extensions` 함수가 뭔지 살펴보자.

```python
def to_jpg(filename):
    return os.path.splitext(filename)[0] + ".jpg"                 # <- line 2

def replace_extensions(csv):
    csv.loc[:,"file_name"] = csv.loc[:,"file_name"].apply(to_jpg) # <- line 5
    return csv
```
* **line 5**은 file_name 칼럼 전체에 `to_jpg`이라는 함수를 apply하는 코드다.
* **line 2**는 각 file_name 값에서 확장자에 해당하는 스트링을 strip하고 대신 `.jpg`를 붙여서 리턴한다.

파일명 확장자를 통일했으니, 이제 본격적으로 오염된 데이터를 걸러내보자.


### 2) 중복 및 모호한 데이터 걸러내기

이번엔 중복되거나 모호한 값을 지닌, 오염된 데이터들을 걸러내는 작업을 할 차례다.

위 예시 이미지처럼 아예 **파일명과 라벨링값이 동일한 데이터들**이 존재하기도 했고,

**파일명은 같은데 다르게 라벨링된 데이터들**이 존재하기도 했다.

`remove_duplicate_rows()`와 `remove_ambiguous_rows()`라는 함수의 파라미터로 넣어 처리했다.

```python
raw_train = remove_duplicate_rows(raw_train)
len(raw_train)
```
<img src="https://user-images.githubusercontent.com/18097984/136975090-a507514d-70c6-40cc-86eb-a12d2797423e.png" width="45%" alt="screenshot" />

```python
raw_train = remove_ambiguous_rows(raw_train)
len(raw_train)
```
<img src="https://user-images.githubusercontent.com/18097984/136980357-b9065629-34e5-4ea0-a8ba-110cb13bf86f.png" width="45%" alt="screenshot" />

실행결과, 데이터 수가 총 415,707개 -> 413,682개 -> 409,470개로 감소한 것을 볼 수 있다.

그럼 이 두 함수가 뭔지 살펴보자.
```python
# 파일명과 라벨들이 duplicate인 row마다 마지막 하나만 살리고 다 없애기
def remove_duplicate_rows(csv):
    cols = list(csv.columns[1:])
    duplicates_removed = csv.drop_duplicates(subset=cols, keep='last')        # <- line 4
    return duplicates_removed
    
# 파일명만 duplicate하고 라벨은 달라서 모호한 row들은 전부 없애버리기
def remove_ambiguous_rows(csv):
    ambiguous_removed = csv.drop_duplicates(subset=['file_name'], keep=False) # <- line 9
    return ambiguous_removed
```
두 함수 다 pandas의 `drop_duplicates` 함수를 활용해 중복되는 데이터를 처리했다.
* 완전히 동일한 데이터일 경우, 그 중 한 개는 살릴 수 있으니까 **line 4**에서 `keep` 옵션을 `'last'`로 지정했다.
* 파일명은 동일한데 평가된 라벨이 다를 경우, 그 중 어떤 데이터가 실제 정답인지 알 길이 없으므로, **line 9**에서는 `keep` 옵션을 `False`로 지정해 살리는 데이터 없이 전부 삭제했다.

### 3) train과 valid 사이 중복 데이터 또 걸러내기

끝난 줄 알았더니 또 다른 문제를 발견했다.

`raw_valid`에도 여태 얘기한 전처리 코드를 똑같이 진행한 뒤 넘어갔는데 뭔가 이상하길래,

```python
# 이미 처리한 train과 valid 사이에는 서로 겹치는 데이터가 없는지 확인
filenames = TOTAL['file_name'].tolist()
seen = set()
uniq = []
duplicates = []
for names in filenames:
    if names not in seen:
        seen.add(names)
        uniq.append(names)
    else:
        duplicates.append(names)
len(duplicates) # 짱 많은 것으로 판명
```

삽질 끝에 또!! train과 valid 데이터셋 사이에 중복되는 값이 존재한다는 것을 뒤늦게 깨달았다.

애초에 이 K-Fashion 데이터셋을 다운받았을 때 train과 valid 데이터셋이 각각 폴더로 구분되어 있길래, 당연히 둘 사이에 중복되는 데이터가 없으리라고 생각했는데 아니었다.

이럴 거면 왜 굳이 이렇게 구분해서 배포했는지 모르겠다... 괜히 전처리 과정만 번거로워진다.

아무튼 그래서 겹치는 데이터를 제거하기 위해, 아래와 같이 `raw_train`과 `raw_valid`을 우선 합친 뒤, 어떻게 처리할지 결정하려고 직접 데이터 테이블을 확인해봤다.

```python
total_data = pd.concat([raw_train, raw_valid])
len(total_data)

# 겹치는 데이터들을 troubles로 명명... 어떻게 처리할지 결정하기 위해 눈으로 직접 데이터 확인
troubles = total_data.loc[total_data['file_name'].isin(duplicates)]
sorted_troubles = troubles.sort_values(['file_name', 'id'], ascending=[True, True]) # 파일명, ID값대로 정렬 후 저장
save_csv(sorted_troubles, "troubles.csv")
```
image TBD

*troubles.csv*

```python
# 엑셀파일과 이미지들을 직접 확인해본 결과...
# 파일명이 겹치는 데이터들 중 나중에 온 ID값을 살리면 되겠다고 판단.
sorted_total = total_data.sort_values(['file_name', 'ID'], ascending=[True, True])
total_without_duplicates = sorted_total.drop_duplicates(subset=['file_name'], keep='last')
len(total_without_duplicates)
```

```python
save_csv(total_without_duplicates, "../../input/preprocessed_TOTAL_dataset.csv")
```


그 다음엔 정말로 이 데이터셋에 나온 파일명 중 누락된 이미지 또한 없는지 확인한다

```python
data = total_without_duplicates
filenames = data['file_name'].tolist()
target_path = os.path.join("..", "..", "input", "ALL_IMAGES/") # 모든 원천데이터(=이미지파일)가 담긴 폴더 경로
candidates = [target_path + name for name in filenames]
len(candidates)
```
```python
data_without_images = []
for path in tqdm(candidates):
    isExist = os.path.exists(path)
    if not isExist:
        print("OH NO.")
        data_without_images.append(path)
    else:
        print("clear!")
```

실행결과 이미지 TBD

*실행 결과 다행히 누락된 이미지는 없는 것으로 보인다.*

Now we are REALLY good to go!!!!

이제 전처리 과정을 다 마쳤으니 본격적으로 train, valid, test 데이터셋으로 각자 쪼개는 작업을 진행해보자.

끝

### 참고
* [How can I replace (or strip) an extension from a filename in Python?](https://stackoverflow.com/questions/3548673/how-can-i-replace-or-strip-an-extension-from-a-filename-in-python)
* [Pandas - Indexing and selecting data](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#)
