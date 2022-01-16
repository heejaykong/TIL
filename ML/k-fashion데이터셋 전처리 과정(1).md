**2021. 09. 05. 기록**

# K-Fashion 이미지 데이터셋을 전처리해 보자 (1) - Data Reduction 편

AI 허브에서 구한 [K-Fashion 이미지 데이터셋](https://aihub.or.kr/aidata/7988)이라는 데이터셋을 전처리하는 대장정을 공유하려고 한다.

## 1. 배경설명

> 포스트 순서:
> 
> **[K-Fashion 이미지 데이터셋을 전처리해 보자 (1) - Data Reduction 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(1).md) <- You are here**
> 
> [K-Fashion 이미지 데이터셋을 전처리해 보자 (2) - Data Cleaning 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(2).md)
> 
> [K-Fashion 이미지 데이터셋을 전처리해 보자 (3) - Generate Train, Validation, Test 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(3).md)
> 
> [K-Fashion 이미지 데이터셋을 전처리해 보자 (4) - Move Image 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(4).md)

졸업작품으로 딥러닝 모델로 사용자의 옷차림을 분석해 날씨와 적합한지 판단해주는 웹 애플리케이션을 만들기로 했다.

<img src="https://user-images.githubusercontent.com/18097984/136150543-220cac34-e572-46f3-9966-c1e30ba389fc.png" width="75%" alt="screenshot"/>

*서비스 개요는 대충 이런 식이다*

위 이미지의 Model 부분에 해당하는 4개의 딥러닝 모델들 중 나는 **상의** 모델을 맡았다.

사용자가 옷차림 이미지를 입력하면 이미지에 나타난 상의의 카테고리와 소매기장을 반환해주는,

**Multi-label image classification** 모델을 만들어야 했다.

모델학습에 활용할 데이터셋으로써 위에 언급한 K-Fashion 이미지 데이터셋을 선택한 후로는, 전처리를 어떻게 할지 고민했다.

결론적으로 내가 진행한 데이터전처리 과정은 아래처럼 4단계로 나눌 수 있다.

1. Data Reduction: 전체 데이터 중 상의 모델학습에 필요한 정보만 뽑아 하나의 csv 파일로 만들기
2. Data Cleaning: 오염된 데이터들 걸러내기
3. Generate Train, Validation, Test: 원하는 양만큼 train, valid, test 데이터 각각 생성하기
4. Move Image: 데이터셋에 해당하는 이미지들 원하는 파일로 옮기기

(각 단계 영문타이틀은 그냥 편의상 구분을 위해 내가 이름 붙였을 뿐, 실제 동일한 용어가 존재한다면 그 뜻과 혼동하지 말자 ㅎ)

이번 편에서는 **1. Data Reduction** 단계를 다뤄보겠다.


## 2. 본문

### 1) 데이터셋 확인

데이터셋을 다운받자마자 볼 수 있는 날것의 구조 그대로는 다음과 같이 생겼다

```
K-FASHION-DATASET
├───Training
            ├───라벨링데이터
                        ├───레트로
                                ├───(116)IMG_1.json
                                ├───(117)IMG_1.json
                                ├───...
                        ├───...
                        ├───히피
                        └───힙합
            └───원천데이터
                        ├───레트로
                                ├───(116)IMG_1.jpg
                                ├───(117)IMG_1.jpg
                                ├───...
                        ├───...
                        ├───히피
                        └───힙합
└───Validation
            ├───라벨링데이터
                        ├───레트로
                                ├───(121)IMG_1.json
                                ├───(211)IMG_1.json
                                ├───...
                        ├───...
                        ├───히피
                        └───힙합
            └───원천데이터
                        ├───레트로
                                ├───(121)IMG_1.jpg
                                ├───(211)IMG_1.jpg
                                ├───...
                        ├───...
                        ├───히피
                        └───힙합
```

*(파일 경로에 한글이 포함돼있으면 추후에 번거로운 일이 발생할 수 있으니 추후에 `라벨링데이터`는 `labels`로, `원천데이터`는 `images`로 디렉토리명을 바꿨다.)*

라벨링데이터는 json 형태다. 예시로 하나의 라벨링데이터를 직접 까서 들여다보자.

아래는 순서대로 **`(116)IMG_1.jpg`** 원천데이터(=이미지)와, 그에 해당하는 라벨링데이터 파일이다.

__(116)IMG_1.jpg__

<img src="https://user-images.githubusercontent.com/18097984/132459633-f87148e3-7c03-4102-b97d-af253cab48b3.jpg" alt="image" width="40%"/>

__(116)IMG_1.json__
```json
{
  "이미지 정보": {
    "이미지 식별자": 703911,
    "이미지 높이": 1066,
    "이미지 파일명": "(116)IMG_1.jpg",
    "이미지 너비": 800
  },
  "데이터셋 정보": {
    "파일 생성일자": "2020-10-29 12:42:59",
    "데이터셋 상세설명": {
      "렉트좌표": {
        "아우터": [
          {}
        ],
        "하의": [
          {}
        ],
        "원피스": [
          {}
        ],
        "상의": [
          {
            "X좌표": 243.5,
            "Y좌표": 478.32,
            "가로": 239,
            "세로": 322
          }
        ]
      },
      "폴리곤좌표": {
        "아우터": [
          {}
        ],
        "하의": [
          {}
        ],
        "원피스": [
          {}
        ],
        "상의": [
          {
            "Y좌표44": 508.809,
            "Y좌표43": 499.812,
            ...(길어서 생략)
            "Y좌표18": 767.712
          }
        ]
      },
      "라벨링": {
        "스타일": [
          {
            "스타일": "레트로",
            "서브스타일": "페미닌"
          }
        ],
        "아우터": [
          {}
        ],
        "하의": [
          {}
        ],
        "원피스": [
          {}
        ],
        "상의": [
          {
            "색상": "네이비",
            "서브색상": "베이지",
            "카테고리": "블라우스",<- 학습에 필요한 정보
            "소매기장": "긴팔",   <- 학습에 필요한 정보
            "소재": [
              "우븐"
            ],
            "프린트": [
              "플로럴"
            ],
            "핏": "루즈"
          }
        ]
      }
    },
    "파일 번호": 703911,
    "파일 이름": "(116)IMG_1.jpg"
  }
}
```
이런 식이다.

우리가 원하는 상의모델을 구축하기 위해서는 `데이터셋 정보 > 데이터셋 상세설명 > 라벨링 > 상의` 하위에 있는 데이터들 중에서도,

`카테고리` 정보와 `소매기장` 정보만 클래스로써 가져오는 것이 좋겠다.


### 2) 상의모델에 필요한 정보만 뽑아오기

지금부터 본격적으로 모델학습에 필요한 데이터셋을 준비할 것이다.

다운받은 데이터셋의 특성상 Training 폴더와 Validation 폴더가 나뉘어 있는데,

지금부터의 코드 예시는 편의상 Training 관련 코드만 적었다는 것을 참고하자. 전체코드는 맨 마지막에 첨부했다.

Multi-label Image classification 딥러닝 튜토리얼들을 찾아보니 공통적으로 모델학습을 위해서는,

학습에 쓰일 데이터셋이 전부 담긴 **하나의 csv파일**이 있어야 하는 것으로 보였다.

<img src="https://user-images.githubusercontent.com/18097984/136779567-bd61f05c-3eaf-4dff-90b3-b5f467bd6407.png" width="50%" alt="screenshot" />

내 데이터셋의 경우, 이렇게 각각의 스타일 폴더(레트로, ... 히피, 힙합) 아래에 나뉘어 담긴 json파일들을 전부 담아 하나의 csv파일로 변환해야 했다.

그래서 아래처럼 glob 함수를 이용해 모든 json파일을 담아왔다. 

```python
# "라벨링데이터" 하위에 있는 모든 json파일 읽어오기
base = "../../../" # 다운받은 k-fashion 데이터셋의 기본 경로 정의
train_label_files = glob(os.path.join(base, "k-fashion-dataset", "Training", "labels", "*", "*"), recursive=True)
```
`train_label_files`의 길이를 확인해보면, Training 폴더에 해당하는 전체 데이터양이 약 70만개인 것을 볼 수 있다.

<img src="https://user-images.githubusercontent.com/18097984/136776103-329bc8bc-27bb-4313-936c-3d42d18552c0.png" width="35%" alt="screenshot" />

그 다음엔 아래와 같이 `generate_csv_from_json_files()` 이라는 함수의 파라미터로 넣었다.
```python
reduced_train = generate_csv_from_json_files(train_label_files)
```
실행한 모습은 아래와 같다.

상의에 해당하는 `카테고리`나 `소매기장` 정보가 없는 데이터는 건너뛴 결과, train 데이터의 총량이 약 40만개로 줄었다.

<img src="https://user-images.githubusercontent.com/18097984/136810696-f3e8cddc-a312-45fe-b8d8-f7da44f07f78.png" width="70%" alt="screenshot" />

그렇다면, `generate_csv_from_json_files()` 함수를 어떻게 작성했는지 까서 확인해보자.

```python
# 읽어온 각각의 json을 필요한 정보만 뽑아서 row 하나로 만들기
def generate_csv_from_json_files(files):
    TEMP_CSV_PATH = "temp.csv"
    data = []
    for single_file in tqdm(files):                                     # <- line 5
        with open(single_file, 'r', encoding='utf-8') as f:
            # Use 'try-except' to skip files that may be missing data
            try:
                json_file = json.load(f)                                # <- line 9
                file_id = json_file["데이터셋 정보"]["파일 번호"]
                file_name = json_file["데이터셋 정보"]["파일 이름"]
                category = json_file["데이터셋 정보"]["데이터셋 상세설명"]["라벨링"]["상의"][0]["카테고리"]
                sleeve_length = json_file["데이터셋 정보"]["데이터셋 상세설명"]["라벨링"]["상의"][0]["소매기장"]
                data.append([file_id, file_name, category, sleeve_length])     # <- line 14
            except KeyError:
                continue
    # Add header
    data.insert(0, ['id', 'file_name', 'category', 'sleeve_length'])
    print("\nREADING EACH JSON FILE ALL DONE! Length of data is", len(data))

    # csv로 변환
    print("Saving data as CSV file...")
    with open(TEMP_CSV_PATH, "w", encoding='utf-8', newline="") as f:   # <- line 23
        writer = csv.writer(f)
        writer.writerows(data)
    print(f"CSV saved in {TEMP_CSV_PATH}")

    # one-hot 인코딩 하기
    csv_file = pd.read_csv(TEMP_CSV_PATH, encoding='utf-8')             # <- line 29
    print("one-hot encoding...")
    targets = csv_file.columns[2:]
    final = pd.get_dummies(csv_file, columns=targets, prefix="", prefix_sep="")

    # 기존에 사용한 파일 이제 무쓸모니까 삭제하기
    if(os.path.exists(TEMP_CSV_PATH) and os.path.isfile(TEMP_CSV_PATH)):
        os.remove(TEMP_CSV_PATH)
        print(f"{TEMP_CSV_PATH} file deleted")
    else:
        print(f"{TEMP_CSV_PATH} file not found")
    
    print("ALL DONE!")
    return final
```
* **line 5**의 루프문을 보자. 학습에 필요한 정보를 각 json 파일마다 뽑아내는 작업을 진행한다.

* **line 9~14**는 그 필요한 정보(image id, file name, 클래스 정보)를 각각 data라는 리스트에 append한다.

* data에 전부 append하고 나면 header를 삽입해 준 뒤, data의 length를 출력해준다.

* **line 23**에서 정해진 경로에 one-hot 인코딩을 위해 임시적으로 data를 csv파일로 만들어 저장시킨다.

* 그렇게 만들어진 csv파일을 **line 29**에서 다시 불러 pandas의 get_dummies 함수를 활용해 one-hot 인코딩을 진행한다.

* 마지막으로, 임시 temp.csv파일을 삭제하고 난 뒤, one-hot 인코딩을 거친 dataframe을 리턴한다.


이렇게 리턴한 dataframe을 할당받은 `reduced_train`을 확인해보자.

<img src="https://user-images.githubusercontent.com/18097984/136813911-76eb9bf4-5e04-422c-ab91-123373157f69.png" width="95%" alt="screenshot" />

*각 클래스 정보가 0 또는 1로 표시되도록 one-hot 인코딩을 거친 상태임을 직접 확인할 수 있다.*

이러면 이제 범주형 데이터가 아닌, yes or no로만 값이 이뤄지는 데이터셋이 되므로, classification으로써 학습시킬 수 있게 된다.

학습을 위한 데이터셋 준비가 다 끝났으니 이제 `save_csv`라는 함수를 통해 이 dataframe을 원하는 경로에 저장해주자.

파일명은 `raw_{데이터종류}.csv` 형식으로 지정했다.
```python
RAW_TRAIN_PATH = os.path.join("..", "..", "input", "raw_train.csv")
save_csv(reduced_train, RAW_TRAIN_PATH)
```
<img src="https://user-images.githubusercontent.com/18097984/136815052-d74e79be-7fc0-4b8a-acb3-38b2777b67d8.png" width="65%" alt="screenshot" />

`save_csv` 함수는 별거 없다. 데이터에 한글이 포함되어 있어서 encoding 옵션을 매번 넣는게 번거로워 함수로 만들었을 뿐이다.
```python
def save_csv(csv, path):
    csv.to_csv(path,
               index = False,
               encoding='utf-8-sig')
    print(f"CSV Saved in {path}")
```

여기까지가 train 데이터셋의 처리 과정이었다.

이와 똑같은 과정을 Validation 폴더 아래의 데이터에게도 진행하면 train과 valid 데이터셋 준비가 각각 완료된다.

<img src="https://user-images.githubusercontent.com/18097984/136834983-625c3f8c-e433-431c-95f2-da34c2b34e0c.png" width="25%" alt="screenshot" />

*생성된 각각의 데이터셋 파일*


## 3. 전체 코드 및 마무리

위 설명에 나온 script들을 포함한 **전체 코드**를 덧붙이며 마무리하고자 한다.

```python
# 1._data_cleaning.ipynb
```
```python
from tqdm import tqdm
from glob import glob
import pandas as pd
import json
import csv
import os
```
```python
# 읽어온 각각의 json을 필요한 정보만 뽑아서 row 하나로 만들기
def generate_csv_from_json_files(files):
    TEMP_CSV_PATH = "temp.csv"
    data = []
    for single_file in tqdm(files):
        with open(single_file, 'r', encoding='utf-8') as f:
            # Use 'try-except' to skip files that may be missing data
            try:
                json_file = json.load(f)
                file_id = json_file["데이터셋 정보"]["파일 번호"]
                file_name = json_file["데이터셋 정보"]["파일 이름"]
                category = json_file["데이터셋 정보"]["데이터셋 상세설명"]["라벨링"]["상의"][0]["카테고리"]
                sleeve_length = json_file["데이터셋 정보"]["데이터셋 상세설명"]["라벨링"]["상의"][0]["소매기장"]
                data.append([file_id, file_name, category, sleeve_length])
            except KeyError:
                continue
    # Add header
    data.insert(0, ['id', 'file_name', 'category', 'sleeve_length'])
    print("\nREADING EACH JSON FILE ALL DONE! Length of data is", len(data))

    # csv로 변환
    print("Saving data as CSV file...")
    with open(TEMP_CSV_PATH, "w", encoding='utf-8', newline="") as f:
        writer = csv.writer(f)
        writer.writerows(data)
    print(f"CSV saved in {TEMP_CSV_PATH}")

    # one-hot 인코딩 하기
    csv_file = pd.read_csv(TEMP_CSV_PATH, encoding='utf-8') # reading the saved csv file
    print("one-hot encoding...")
    targets = csv_file.columns[2:]
    final = pd.get_dummies(csv_file, columns=targets, prefix="", prefix_sep="")

    # 기존에 사용한 파일 이제 무쓸모니까 삭제하기
    if(os.path.exists(TEMP_CSV_PATH) and os.path.isfile(TEMP_CSV_PATH)):
        os.remove(TEMP_CSV_PATH)
        print(f"{TEMP_CSV_PATH} file deleted")
    else:
        print(f"{TEMP_CSV_PATH} file not found")
    
    print("ALL DONE!")
    return final
```
```python
def save_csv(csv, path):
    csv.to_csv(path,
                index = False,
                encoding='utf-8-sig')
    print(f"CSV Saved in {path}")
```
```python
# "라벨링데이터" 하위에 있는 모든 json파일 읽어오기
base = "../../../"

train_label_files = glob(os.path.join(base, "k-fashion-dataset", "Training", "labels", "*", "*"), recursive=True)
reduced_train = generate_csv_from_json_files(train_label_files)

RAW_TRAIN_PATH = os.path.join("..", "..", "input", "raw_train.csv")
save_csv(reduced_train, RAW_TRAIN_PATH)
```
```python
valid_label_files = glob(os.path.join(base, "k-fashion-dataset", "Validation", "labels", "*", "*"), recursive=True)
reduced_valid = generate_csv_from_json_files(valid_label_files)

RAW_VALID_PATH = os.path.join("..", "..", "input", "raw_valid.csv")
save_csv(reduced_valid, RAW_VALID_PATH)
```
```python
# To be continued... (2._data_cleaning.ipynb)
```

이번 글에서는 K-Fashion 이미지 데이터셋을 우리의 목표에 알맞은 정보들만 빼서 담아내는 자칭 **"1. Data Reduction"** 과정을 살펴보았다.

다음 글에서는 이 데이터셋에서 본격적으로 오염된 데이터들을 걸러내는 **"2. Data Cleaning"** 과정을 알아보자.

끝


### 참고
* [Using Python to Read Multiple JSON Files and Export Values to a CSV](https://blr.design/blog/python-multiple-json-to-csv)
