**2021. 09. 05. 기록**

# K-Fashion 이미지 데이터셋을 전처리해 보자 (3) - Generate Train, Validation, Test 편

## 1. 배경설명

> 포스트 순서:
> 
> [K-Fashion 이미지 데이터셋을 전처리해 보자 (1) - Data Reduction 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(1).md)
> 
> [K-Fashion 이미지 데이터셋을 전처리해 보자 (2) - Data Cleaning 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(2).md)
> 
> **[K-Fashion 이미지 데이터셋을 전처리해 보자 (3) - Generate Train, Validation, Test 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(3).md) <- You are here**
> 
> [K-Fashion 이미지 데이터셋을 전처리해 보자 (4) - Move Image 편]()

이전 포스트들에서 설명했듯이 [K-Fashion 이미지 데이터셋](https://aihub.or.kr/aidata/7988)을 활용해,

사용자가 옷차림 이미지를 입력하면 이미지에 나타난 상의의 카테고리와 소매기장을 반환해주는

**Multi-label image classification** 모델을 만들어야 했다.

데이터 전처리 과정은 아래처럼 4단계로 나눴다.

1. Data Reduction: 전체 데이터 중 상의 모델학습에 필요한 정보만 뽑아 하나의 csv 파일로 만들기
2. Data Cleaning: 오염된 데이터들 걸러내기
3. Generate Train, Validation, Test: 원하는 양만큼 train, valid, test 데이터 각각 생성하기
4. Move Image: 데이터셋에 해당하는 이미지들 원하는 파일로 옮기기

이번 편에서는 **3. Generate Train, Validation, Test** 단계를 다뤄보겠다. **전체코드는 마지막에 첨부**했다.

## 2. 본문

이전 포스트에서는 raw_train과 raw_valid 데이터셋을 합친 뒤 중복 데이터들을 처리한 `preprocessed_total_dataset.csv` 파일을 하나 만들었다.

<img src="https://user-images.githubusercontent.com/18097984/137585368-2aa2f47d-290c-4480-987b-1c1ce19ee6aa.png" width="35%" alt="screenshot" />

이번에는 `preprocessed_total_dataset.csv`에서 train, valid, test 데이터셋을 각각 뽑아내는 과정을 아래 순서대로 거칠 것이다.

1. 원하는 양만큼 랜덤데이터 뽑기
2. 분포도 확인하며 이 데이터셋의 한계점 깨닫기

그러면 시작해보자...

### 1. 원하는 양만큼 랜덤데이터 뽑기

### 2. 분포도 확인하며 이 데이터셋의 한계점 깨닫기

## 3. 전체코드 및 마무리

전체코드는 다음과 같다.

```python
import os
import pandas as pd
```

```python
# 각각 원하는 크기로 sub dataset 랜덤으로 뽑아내기
def randomlySplitToDatasets(csv, TRAIN_SIZE, VALID_SIZE, TEST_SIZE):
    TRAIN_SIZE = TRAIN_SIZE
    VALID_SIZE = VALID_SIZE
    TEST_SIZE = TEST_SIZE
    TOTAL_SIZE = TRAIN_SIZE + VALID_SIZE + TEST_SIZE

    total_samples = csv.sample(n=TOTAL_SIZE) # 겹치지 않게 랜덤셀렉

    train = total_samples[:TRAIN_SIZE]
    valid = total_samples[TRAIN_SIZE : TRAIN_SIZE + VALID_SIZE]
    test = total_samples[TRAIN_SIZE + VALID_SIZE:]
    
    print(len(train), len(valid), len(test))
    return [train, valid, test]
```

```python
def showDistribution(csv):
    # '카테고리'와 '소매기장' 칼럼들을 다루기 위해 우선 칼럼명을 각각 list에 넣어준다
    classes = list(csv.columns[2:])
    categories = classes[:7]
    sleeves = classes[7:]
    
    # '카테고리'(니트웨어, 브라탑, 블라우스, 셔츠...)와 '소매기장'(긴팔, 7부, 반팔...) 칼럼으로
    # 만들 수 있는 모든 조합을 각각 sub dataframe으로 만들어 list에 저장한다.
    DATAFRAME_ARR = []
    combinations = []
    for i, category in enumerate(categories):
        for j, sleeve in enumerate(sleeves):
            combinations.append(sleeve + " " + category)
            sub = csv[(csv[sleeve] == 1) & (csv[category] == 1)]
            DATAFRAME_ARR.append(sub)
    
    for i, DATAFRAME in enumerate(DATAFRAME_ARR):
#         print(combinations[i], ":", len(DATAFRAME))
        print('{:10}'.format(combinations[i]), ":", len(DATAFRAME))
```

```python
# 파일로 저장하기
def save_csv(csv, path):
    csv.to_csv(path, index = False, encoding="utf-8-sig")
    print("csv file is saved in", path)
```

```python
base = "../../input/"
csv = os.path.join(base, "preprocessed_total_dataset.csv")
data = pd.read_csv(csv, encoding='utf-8')
print(len(data))
```

```python
train, valid, test = randomlySplitToDatasets(data, 30000, 10000, 10000)
# showDistribution(test)
# showDistribution(valid)
# showDistribution(test)
```

```python
TRAIN_PATH = os.path.join(base, "train_data.csv")
VALID_PATH = os.path.join(base, "valid_data.csv")
TEST_PATH = os.path.join(base, "test_data.csv")

save_csv(train, TRAIN_PATH)
save_csv(valid, VALID_PATH)
save_csv(test, TEST_PATH)
```

```python
# To be continued... (4._image_move.ipynb)
```

### 참고
* []()
* []()
