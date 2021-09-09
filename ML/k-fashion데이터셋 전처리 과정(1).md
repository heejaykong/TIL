**2021. 09. 05. 기록**

# K-Fashion 이미지 데이터셋을 전처리해 보자 (1) - Data Reduction 편

AI 허브에서 구한 K-Fashion 이미지 데이터셋이라는 데이터셋을 전처리하는 대장정을 공유하려고 한다.

### 배경설명

졸업작품 어쩌구

날씨에 맞는 코디인지 아닌지 알려주는 서비스

우선 이미지를 넣으면 어떤 옷을 입고 있는지 알려주는 모델을 먼저 만들고 싶었다

그래서 이 K-Fashion 이미지 데이터셋을 활용하기로 결정.

모델을 구축하기 위해서는 먼저 모델에 맞는 데이터셋을 만들기 위해 데이터전처리를 진행해야 함.

내가 진행한 데이터전처리 과정은 아래처럼 다섯 단계로 나눌 수 있다.

1. Data Reduction: 모든 라벨링 데이터들을 모델학습에 사용가능한 csv 형식으로 먼저 만들기
2. Data Cleaning: 전체 데이터 중, 오염된 데이터들 걸러내기
3. Generate Train, Validation, Test: 원하는 양만큼 train, valid, test 데이터 생성하기
5. Move Image: 데이터셋에 해당하는 이미지들 원하는 파일로 옮기기

(각 단계 타이틀은 그냥 편의상 구분을 위해 내가 이름 붙였을 뿐, 실제로 사용하는 용어는 아닐수도 있고 설령 현업에서 사용하는 용어와 겹치는 게 있다해도 그 뜻은 아닐 것이다.)

이번 편에서는 **1. Data Reduction 단계**를 다뤄보겠다.


### 본문

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

파일 경로에 한글이 포함돼있으면 추후에 번거로운 일이 발생할 수 있으니 `라벨링데이터`는 `labels`, `원천데이터`는 `images`로 파일명을 영어로 바꿨다.

라벨링데이터는 json 형태다. 예시로 하나의 json 파일을 직접 까서 들여다보자. 아래는 (116)IMG_1.jpg에 해당하는 json 파일이다.

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
            ...(엄청 길어서 생략)
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
            "카테고리": "블라우스",
            "소매기장": "긴팔",
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

image classification 딥러닝 튜토리얼들을 보니 데이터셋 전체가 담긴 하나의 csv파일이 있어야 하는 것으로 보였다.

즉 여기 각각의 스타일 파일에 나뉘어져 담긴 모든 json 파일들을 각각 읽어와서 하나의 csv파일에 담고 싶었다.

다음과 같이 glob 함수를 이용해 모든 json파일을 담아왔고

```python
# "라벨링데이터" 하위에 있는 모든 json파일 읽어오기 (참고: https://blr.design/blog/python-multiple-json-to-csv)
base = "../../" # 다운받은 k-fashion 데이터셋으로 가는 기본 경로 정의
train_label_files = glob(os.path.join(base, "k-fashion-dataset", "Training", "labels", "*", "*"), recursive=True)
valid_label_files = glob(os.path.join(base, "k-fashion-dataset", "Validation", "labels", "*", "*"), recursive=True)
```

`generate_csv_from_json_files()` 이라는 커스텀 함수의 파라미터로 넣었다. 이러면 정해진 경로에 temp.csv라는, 전처리당할 준비가 된 csv파일이 저장된다

```python
rd.generate_csv_from_json_files(train_label_files)
```
<img src="https://user-images.githubusercontent.com/18097984/132627140-f1da7e9f-6624-488e-a273-99dd8b29d7e3.png" width="70%" alt="screenshot" />

그럼 `generate_csv_from_json_files()` 함수가 어떻게 생겼는지 까보자

```python
# 읽어온 각각의 json을 필요한 정보만 뽑아서 row 하나로 만들기
def generate_csv_from_json_files(files):
    data = []
    for single_file in tqdm(files):
        with open(single_file, 'r', encoding='utf-8') as f:
            # Use 'try-except' to skip files that may be missing data
            try:
                json_file = json.load(f)
                file_id = json_file["데이터셋 정보"]["파일 번호"]
                file_name = json_file["데이터셋 정보"]["파일 이름"]
                top = json_file["데이터셋 정보"]["데이터셋 상세설명"]["라벨링"]["상의"][0] # 상의만 가져올 것임
                data.append([file_id, file_name, top])
            except KeyError:
                print(f'Skipping {single_file}')
    # Add header
    data.insert(0, ['ID', 'File Name', 'Top'])
    print("\nREADING EACH JSON FILE ALL DONE! Length of data is", len(data))

    # csv로 변환
    print("Saving data as CSV file...")
    with open(TEMP_CSV_PATH, "w", encoding='utf-8', newline="") as f:
        writer = csv.writer(f)
        writer.writerows(data)
    print(f"CSV saved in {TEMP_CSV_PATH}")
```

line 2의 루프문에서 각각의 json file마다 필요한 처리를 할 것이다.

line 7~12까지는 우리에게 필요한 정보(image id, file name, 상의 관련 전체정보)를 각각 뽑아주고, 이를 data에 append한다.

data에 전부 append하고 나면 header를 삽입해주고, 이제 line 21에서 정해진 경로에 임시적으로 data를 csv파일로 만들어 저장시킨다.


그렇게 저장된 임시파일인 temp.csv를 확인해보자. json객체가 아예 통째로 저장된 상태이기 때문에 literal_eval을 이용해서 불러온다.

```python
data = pd.read_csv(rd.TEMP_CSV_PATH,
                    converters = {'Top': literal_eval},
                    encoding='utf-8') # reading the csv file
data
```

<img src="https://user-images.githubusercontent.com/18097984/132625515-17334b5b-e0fe-49bf-a437-a407d9f01987.png" width="75%" alt="screenshot"/>

이렇게 생겼다. Top이라는 칼럼 안에 json객체가 통째로 들어있는 것을 볼 수 있다.

그러면 이제 불러온 이 dataframe을 `reduce_data()`라는 커스텀함수에 또 돌려보자.

이 함수는 json 객체를 일일이 칼럼으로 flatten시키고, 우리에게 필요한 칼럼만 남기고, 마지막으로 one-hot 인코딩까지 마친 dataframe을 리턴해준다


```python
reduced_data = rd.reduce_data(data)
```

<img src="https://user-images.githubusercontent.com/18097984/132627817-07245f4d-8e83-4f38-98e6-dddaa9da1dea.png" width="70%" alt="screenshot" />

넘어가기 전에 함수가 어떻게 생겨먹었는지 들여다보자

```python
def reduce_data(csv):
    # csv칼럼 내부 json 객체 flatten하기
    # https://stackoverflow.com/questions/39899005/how-to-flatten-a-pandas-dataframe-with-some-columns-as-json
    print("flattening columns...")
    TOP = pd.json_normalize(csv["Top"])
    flattened = csv[['ID', 'File Name']].join(TOP)

    # 우리 모델에 필요한 칼럼만 남기기
    print("dropping columns we don't need...")
    dropped = flattened[['ID', 'File Name', '카테고리','소매기장']]

    # one-hot 인코딩 하기
    print("one-hot encoding...")
    pd.set_option('display.max_columns', None)
    targets = dropped.columns[2:]
    final = pd.get_dummies(dropped, columns=targets, prefix_sep="_")

    # 기존에 사용한 파일 이제 무쓸모니까 삭제하기
    if(os.path.exists(TEMP_CSV_PATH) and os.path.isfile(TEMP_CSV_PATH)):
        os.remove(TEMP_CSV_PATH)
        print(f"{TEMP_CSV_PATH} file deleted")
    else:
        print(f"{TEMP_CSV_PATH} file not found")
    
    print("ALL DONE!")
    return final
```

line 5에서 json_normalize를 사용해 json객체 값들을 전부 칼럼으로 변환시켜줬고

line 10에서 우리에게 필요한 칼럼들(카태고리, 소매기장)만 남긴 dropped라는 dataframe을 만들었다. 날씨와 코디를 연관시키려면 긴팔 블라우스, 처럼 소매기장과 카테고리만 있으면 될 것 같았다.

line 16에서 pandas의 get_dummies 함수를 사용해 one-hot 인코딩을 진행했다. 이러면 이제 범주형 데이터가 아닌 0과 1로만 값이 이뤄지는 데이터셋이 되므로, classification으로써 학습시킬 수 있게 된다.

line 18~23는 더이상 필요 없어진 기존의 temp.csv 파일을 삭제하는 코드다


짠~ 그리하여 생성된 최종 모습을 봐보자

image TBD
![image](https://user-images.githubusercontent.com/18097984/132628097-f96126d2-fedd-47bb-9497-fc4b77149409.png)

여기까지가 train 데이터셋의 처리 과정이었다. 이와 똑같은 과정을 valid 데이터셋에게도 똑같이 진행한다.

첫번째 전처리를 마친 이 각각의 train, valid 데이터셋을 원하는 경로에 저장해주자. 나는 raw_{데이터명}.csv라는 이름으로 저장했다.

```python
raw_train_path = os.path.join("..", "input", "raw_train.csv")
rd.save_csv(reduced_data, raw_train_path)
```
<img src="https://user-images.githubusercontent.com/18097984/132629556-9c6d07fc-baff-4acf-a4f2-1909f0ab2440.png" width="75%" alt="screenshot" />

save_csv()도 커스텀 함수인데, 별거 없다
```python
def save_csv(csv, path):
    csv.to_csv(path,
                index = False,
                encoding='utf-8-sig')
    print(f"CSV Saved in {path}")
```

index 없이 저장하는 옵션, 그리고 한글 값이 포함돼있어서 저장할때 encoding하는 옵션을 매번 신경쓰는 게 번거로워서 그냥 모듈화시켰다.

### 마무리

이번 글에서는 K-Fashion 이미지 데이터셋을 우리의 목표에 알맞은 정보들만 빼내서 담아내는 자칭 **"Data Reduction"** 과정을 살펴보았다.

다음 글에서는 본격적으로 오염된 데이터들을 걸러내는 **"Data Cleaning"** 과정을 알아보자.

끝
