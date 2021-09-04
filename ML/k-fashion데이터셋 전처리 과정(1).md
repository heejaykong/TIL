**2021. 09. 05. 기록**

# K-Fashion 이미지 데이터셋을 전처리해 보자 (1)
### Data Reduction 편

AI 허브에서 구한 K-Fashion 이미지 데이터셋이라는 데이터셋을 전처리하는 대장정을 공유하려고 한다.

**배경설명:**

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


**본문:**

데이터셋을 다운받자마자 볼 수 있는 날것의 상태 그대로는 다음과 같이 생겼다

image TBD

파일 경로에 한글이 포함돼있으면 추후에 번거로운 일이 발생할 수 있으니 우선 다음처럼 파일명을 영어로 바꿨다

image TBD

라벨 데이터는 json형태다. 각각의 json파일을 직접 까서 들여다보자

image TBD

image classification 딥러닝 튜토리얼들을 보니 데이터셋 전체가 담긴 하나의 csv파일이 있어야 하는 것으로 보였다.

즉 여기 각각의 스타일 파일에 나뉘어져 담긴 모든 json 파일들을 각각 읽어와서 하나의 csv파일에 담고 싶었다.

다음과 같이 glob 함수를 이용해 모든 json파일을 담아왔고

code TBD

generate_csv_from_json_files() 이라는 커스텀 함수의 파라미터로 넣었다. 이러면 정해진 경로에 temp.csv라는, 전처리당할 준비가 된 csv파일이 저장된다

code TBD

image TBD(temp.csv 모습)

그럼 generate_csv_from_json_files() 함수를 까보자

code TBD

line n의 루프문에서 각각의 json file마다 필요한 처리를 할 것이다.

line n~ n까지는 우리에게 필요한 정보(image id, file name, 상의 관련 전체정보)를 각각 뽑아주고, 이를 data에 append한다.

data에 전부 append하고 나면 header를 삽입해주고, 이제 line n에서 정해진 경로에 임시적으로 data를 csv파일로 만들어 저장시킨다.


그렇게 저장된 임시파일인 temp.csv를 이제 본격적 전처리를 위해 불러와보자. json객체가 아예 통째로 저장됐기 때문에 literal_eval을 이용한다.

code TBD

불러온 dataframe을 reduce_data()라는 커스텀함수에 돌려보자.

이 함수는 json 객체를 일일이 칼럼으로 flatten시키고, 우리에게 필요한 칼럼만 남기고, 마지막으로 one-hot 인코딩까지 마친 dataframe을 리턴해준다

제대로 까서 확인해보자

code TBD

line n에서 json_normalize를 사용해 json객체 값들을 전부 칼럼으로 변환시켜줬고

line n에서 우리에게 필요한 칼럼들(카태고리, 소매기장)만 남긴 dropped라는 dataframe을 만들었다. 날씨와 코디를 연관시키려면 긴팔 블라우스, 처럼 소매기장과 카테고리만 있으면 될 것 같았다.

line n에서 pandas의 get_dummies 함수를 사용해 one-hot 인코딩을 진행했다. 이러면 이제 범주형 데이터가 아닌 0과 1로만 값이 이뤄지는 데이터셋이 되므로,classification으로써 학습시킬 수 있게 된다.

line n~n는 더이상 필요 없어진 기존의 temp.csv를 삭제하는 코드다


짠 그리하여 생성된 최종 모습을 봐보자

image TBD

크게 두 단계로 나눠진 전처리과정 중 첫번째 단계를 마친 이 각각의 train, valid 데이터셋을 원하는 경로에 저장해주자. 나는 raw_{데이터명}.csv라는 이름으로 저장했다.

save_csv()도 커스텀 함수인데, 별거 없다

code TBD

index 없이 저장하는 옵션, 그리고 한글 값이 포함돼있어서 저장할때 encoding 부분을 매번 신경쓰는 게 번거로워서 이것도 그냥 모듈화시켰다.


이번 글에서는 K-Fashion 이미지 데이터셋을 우리의 목표에 알맞은 정보들만 빼내서 담아내는 자칭 "Data Reduction" 과정을 살펴보았다.

다음 글에서는 본격적으로 오염된 데이터들을 걸러내는 "Data Cleaning" 과정을 알아보자.

끝
