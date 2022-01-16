**2021. 09. 05. 기록**

# K-Fashion 이미지 데이터셋을 전처리해 보자 (4) - Move Image 편

## 1. 배경설명

> 포스트 순서:
> 
> [K-Fashion 이미지 데이터셋을 전처리해 보자 (1) - Data Reduction 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(1).md)
> 
> [K-Fashion 이미지 데이터셋을 전처리해 보자 (2) - Data Cleaning 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(2).md)
> 
> [K-Fashion 이미지 데이터셋을 전처리해 보자 (3) - Generate Train, Validation, Test 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(3).md)
> 
> **[K-Fashion 이미지 데이터셋을 전처리해 보자 (4) - Move Image 편]() <- You are here**

이전 포스트들에서 설명했듯이 [K-Fashion 이미지 데이터셋](https://aihub.or.kr/aidata/7988)을 활용해,

사용자가 옷차림 이미지를 입력하면 이미지에 나타난 상의의 카테고리와 소매기장을 반환해주는

**Multi-label image classification** 모델을 만들어야 했다.

데이터 전처리 과정은 아래처럼 4단계로 나눴다.

1. Data Reduction: 전체 데이터 중 상의 모델학습에 필요한 정보만 뽑아 하나의 csv 파일로 만들기
2. Data Cleaning: 오염된 데이터들 걸러내기
3. Generate Train, Validation, Test: 원하는 양만큼 train, valid, test 데이터 각각 생성하기
4. Move Image: 데이터셋에 해당하는 이미지들 원하는 파일로 옮기기

이번 편에서는 **4. Move Image** 단계를 다뤄보겠다. **전체코드는 마지막에 첨부**했다.

## 2. 본문

이전 포스트에서는 train, valid, test 데이터셋을 각각 랜덤하게 3:1:1 비율로 뽑아냈었다.

<img src="https://user-images.githubusercontent.com/18097984/149661189-dcf480f8-6331-47e3-83f9-74e7433ac564.png" alt="screenshot" width="35%" />

이번에는 각 train, valid, test 데이터셋이 갖고 있는 실제 이미지들을 따로 옮기는 작업을 진행할 것이다.

엄청난 작업을 하는 건 아니고 말그대로 사진만 옮기면 끝이라, 이어서 코드만 설명하겠다.

다음과 같이 copyImages라는 함수를 짰다.

file_name에 해당하는 데이터만 따로 뽑아, 각 파일명마다에 해당하는 복사할곳(origin), 붙여넣을곳(destination) 경로 만들어 준 뒤,

파이썬 shutil의 copy라는 함수를 이용해 for문을 돌려 각각 복사하는 코드다.

```python
# 데이터셋의 파일명과 동일한 이미지파일들을 원하는 곳으로 옮기기
def copyImages(csv, origin_path, destination_path):
    # converting column data to list
    filenames = csv['file_name'].tolist()
    
    # 각 파일명마다 복사할곳(origin), 붙여넣을곳(destination) 경로 만들어주기
    origin_paths      = [os.path.join(origin_path,      filename) for filename in filenames]
    destination_paths = [os.path.join(destination_path, filename) for filename in filenames]
    
    # 복사하기
    for i in tqdm(range(len(origin_paths))):
        try:
            copy(origin_paths[i], destination_paths[i])
        except IOError as e:
            # print("Unable to copy file. %s" % e)
            # print(origin_paths[i])
            continue
```

파라미터로 사용하고자 하는 csv파일, 복사하고자 하는 경로, 그리고 붙여넣고자 하는 경로를 정의해주면 된다.

이제 이 함수가 포함된 전체 코드를 봐보자.

## 3. 전체코드 및 마무리

전체코드는 다음과 같다.

```python
import os
import pandas as pd
from shutil import copy
from tqdm import tqdm
```

```python
# 데이터셋의 파일명과 동일한 이미지파일들을 원하는 곳으로 옮기기
def copyImages(csv, origin_path, destination_path):
    # converting column data to list
    filenames = csv['file_name'].tolist()
    
    # 각 파일명마다 복사할곳(origin), 붙여넣을곳(destination) 경로 만들어주기
    origin_paths      = [os.path.join(origin_path,      filename) for filename in filenames]
    destination_paths = [os.path.join(destination_path, filename) for filename in filenames]
    
    # 복사하기
    for i in tqdm(range(len(origin_paths))):
        try:
            copy(origin_paths[i], destination_paths[i])
        except IOError as e:
            # print("Unable to copy file. %s" % e)
            # print(origin_paths[i])
            continue
```

```python
# reading CSV file
base = "../../input/"
train = pd.read_csv(os.path.join(base, "train_data.csv"))
valid = pd.read_csv(os.path.join(base, "valid_data.csv"))
test = pd.read_csv(os.path.join(base, "test_data.csv"))
```

```python
origin_path = os.path.join(base, "ALL_IMAGES/")
train_destination_path = os.path.join(base, "train_images/")
valid_destination_path = os.path.join(base, "valid_images/")
test_destination_path = os.path.join(base, "test_images/")
```

```python
copyImages(train, origin_path, train_destination_path)
```

```python
copyImages(valid, origin_path, valid_destination_path)
```

```python
copyImages(test, origin_path, test_destination_path)
```

<img src="https://user-images.githubusercontent.com/18097984/149663218-bbb2570f-fc40-4999-8e48-21703619b815.png" alt="screenshot" width="70%" />

<img src="https://user-images.githubusercontent.com/18097984/149664641-6c9a3c8a-cd9b-44d6-92ca-b9c25363cb64.png" alt="screenshot" width="30%" />

<img src="https://user-images.githubusercontent.com/18097984/149664966-d5ad6797-7a79-4a35-855b-fa968e4ae99b.png" alt="screenshot" width="80%" />

_copyImages를 각 train, valid, test에 돌린 결과 위와 같이 이미지파일들이 잘 복사된 걸 볼 수 있다._


#### 드디어 전처리 과정을 마쳤다.

이제 이 데이터를 가지고 모델을 학습시킬 수 있다.

만약 AI허브의 K-Fashion 이미지 데이터셋을 쓰는 사람이 있다면 내 기록이 조금이나마 도움이 되었으면 좋겠다. (물론 전처리하고자 하는 방향이 다를 순 있어도 힌트나마 되었으면 좋겠다.)

인공지능을 공부해본적도, 모델학습이 뭔지도 전처리가 무슨 개념인지도 몰랐던 제로베이스의 내가 처음부터 끝까지 이 과정을 거쳤다는 것이 뿌듯하다 ㅎㅎ

나중에 시간이 된다면 이 데이터셋으로 학습을 시킨 과정도 기록해보고자 한다.

### 참고
* [[youtube]2. 데이터셋의 이해, 데이터 전처리 - K-Fashion AI 경진대회](https://youtu.be/bJqvhBoG3FQ)
