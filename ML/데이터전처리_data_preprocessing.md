**2021.08.13. 기록**
# JSON으로 이뤄진 데이터를 파이썬으로 전처리해보자
### 공공 데이터셋에서 내가 필요한 정보만 뽑아내는 과정 중 배운 것들

졸업작품으로 날씨에 적합한 코디를 추천하는 서비스를 만들어보기 위해, 의상의 이미지와 그에 맞게 라벨링된 json파일을 제공하는 데이터셋을 AI허브에서 구했다. ([K-Fashion 이미지](https://aihub.or.kr/aidata/7988))

사람이 옷을 입은 이미지를 입력했을 때 아우터, 상의, 하의 또는 원피스의 기장, 소매기장, 소재, 또는 색상 등을 알려주는 Multi label image classification model을 만들고 싶었다. 전체적인 실습은 [이 링크](https://www.analyticsvidhya.com/blog/2019/04/build-first-multi-label-image-classification-model-python/)를 참고했다.

해당 튜토리얼을 따라하기 위해서는 하나의 온전하고 아름다운 csv파일을 만들어야 했고, 우리 데이터셋은 다수의 json 파일들이었기 때문에 필요한 정보들만 취해서 csv로 변환하는 전처리 작업이 필요했다.

이번 포스트는 라벨링데이터들(json파일들)을 전처리한 과정을 소개하려 한다.

사실 아무것도 모르는 상태에서 머신러닝을 첨 시도해봐서 이렇게 하는 게 효율적인 방법인진 모르겠다. 어쨌든 one hot 인코딩된 csv파일을 얻는 덴 성공했다.


**학습 내용 요약:**

1. 다수의 JSON파일을 읽으려면 glob 라이브러리를 사용하는 방법이 있다.
2. csv파일에서 JSON 객체를 값으로 갖고 있는, 다수의 칼럼을 flatten하려면 json_normalize와 join을 사용하는 방법이 있다.
3. 데이터를 간소화시키기 위해 일부 칼럼을 drop할 수 있다.
4. list에서 특정 인덱스를 추출한 값으로 칼럼값들을 일괄변경하고 싶으면 str.get() 함수를 사용하는 방법이 있다.
5. 범주형 변수를 0과 1의 데이터로 바꾸려면, 즉 원핫인코딩을 하려면 pandas의 `get_dummies()` 함수를 사용할 수 있다.

**본문:**

1. 다수의 JSON파일을 읽으려면 glob 라이브러리를 사용하는 방법이 있다. ([참고 링크](https://blr.design/blog/python-multiple-json-to-csv/))
    - 사용 예시: 우리 데이터셋은 다음과 같은 구조였는데, json파일들을 전부 읽어서 하나의 csv파일로 export하고 싶었다.
    - 그래서 `files = glob.glob('.../k-fasion-api/Training/라벨링데이터/*/*', recursive=True)` 이런 식으로 읽어들임

    ```
    k-fashion-데이터셋
    └───Training
        └───라벨링데이터
            ├───레트로
                ├───(116)IMG_1.json
                ├───(117)IMG_1.json
                ├───(118)IMG_1.json
                ├───...
            ├───로맨틱
            ├───...
            ├───히피
            └───힙합
    ```


2. csv파일에서 JSON 객체를 값으로 갖고 있는, 다수의 칼럼을 flatten하려면 json_normalize와 join을 사용하는 방법이 있다. ([참고 링크](https://stackoverflow.com/questions/39899005/how-to-flatten-a-pandas-dataframe-with-some-columns-as-json))
    - 사용 예시: 아우터(Outer), 하의(Bottom), 원피스(Dress), 상의(Top) 칼럼마다 JSON 객체가 들어가 있던 상황. 이를 칼럼마다 json_normalize하고 전부 join했음

    ![image](https://user-images.githubusercontent.com/18097984/129445928-487fbeae-66e0-46dc-b5ce-4646a926258c.png)

    _처음 csv파일 읽었을 때 상태(4개의 칼럼에 JSON객체가 값으로 들어가 있는 상태)_

    ```python
    OUTER = pd.json_normalize(train["Outer"]).add_prefix('Outer.')
    BOTTOM = pd.json_normalize(train["Bottom"]).add_prefix('Bottom.')
    DRESS = pd.json_normalize(train["Dress"]).add_prefix('Dress.')
    TOP = pd.json_normalize(train["Top"]).add_prefix('Top.')
    print("FLATTENING ALL DONE.")
    print("now joining...")
    train = train[['ID', 'File Name']].join([OUTER, BOTTOM, DRESS, TOP])
    print("ALL DONE!")
    ```

    ![image](https://user-images.githubusercontent.com/18097984/129445979-4dce4644-e3bb-4681-8457-164959f864cf.png)

    _코드를 실행한 결과, 성공적으로 flatten되어 칼럼 수가 43개가 된 것을 볼 수 있다._


3. 데이터를 간소화시키기 위해 일부 칼럼을 drop할 수 있다.
    - 사용 예시: 우리 모델에 필요한 칼럼들(기장, 소매기장, 소재, 카테고리 등)만 사용하기 위해 나머지 칼럼들 전부 drop

    ![image](https://user-images.githubusercontent.com/18097984/129445989-9baf958b-329e-490c-8164-92561237cc58.png)


4. list에서 특정 인덱스를 추출한 값으로 칼럼값들을 일괄변경하고 싶으면 str.get() 함수를 사용하는 방법이 있다. ([참고 링크](https://stackoverflow.com/questions/42741453/iterating-over-a-column-and-replacing-a-value-with-an-extracted-string-pandas))
    - 사용 예시: 테이블에서 각 소재 칼럼마다 list의 0번째 인덱스를 추출해야 했다.
    - `train['어쩌구.소재'] = train['어쩌구.소재'].str.get(0)` 코드로 각 칼럼마다 실행해줬다.

    ![image](https://user-images.githubusercontent.com/18097984/129446006-0e604b3a-97ca-4c3f-bf66-7e30070bd713.png)

    _list형태 값을 지닌 문제의 칼럼들(캡쳐에 안나왔을 뿐 Outer.소재, Bottom소재, Dress.소재, Top.소재 전부 해당)_

    ```python
    train['Outer.소재'] = train['Outer.소재'].str.get(0)
    train['Bottom.소재'] = train['Bottom.소재'].str.get(0)
    train['Dress.소재'] = train['Dress.소재'].str.get(0)
    train['Top.소재'] = train['Top.소재'].str.get(0)
    train.head()
    ```

    ![image](https://user-images.githubusercontent.com/18097984/129446015-23cfc0e7-c0fb-4ee6-af27-a308ee39df16.png)

    _위 코드 실행 후 결과 소재 칼럼들의 list에서 추출된 string만 남았다_


5. 범주형 변수를 0과 1의 데이터로 바꾸려면, 즉 원핫인코딩을 하려면 pandas의 `get_dummies()` 함수를 사용할 수 있다.([참고 링크](https://opentutorials.org/course/4570/28987))
    - 사용 예시: 우리 데이터의 종속변수는 전부 범주형 데이터이기 때문에 데이터ID와 이미지파일명 칼럼을 제외한 모든 칼럼에 원핫인코딩을 시행했다.

    ```python
    columns = train.columns[2:] # 0번째(ID), 1번째(파일명) 칼럼은 인코딩 대상에서 제외
    train = pd.get_dummies(train, columns=columns, prefix_sep=".")
    print("one-hot encoded.")
    ```

    ![image](https://user-images.githubusercontent.com/18097984/129446024-1033c75f-9b37-4056-9689-0afe4b45280c.png)

    _코드 실행 결과 범주형 데이터들이 전부 0과 1의 데이터로 바뀌었다._
 
 
 ### 참고
 * [AI허브 K-Fashion 이미지 데이터셋](https://aihub.or.kr/aidata/7988)
 * [Build your First Multi-Label Image Classification Model in Python](https://www.analyticsvidhya.com/blog/2019/04/build-first-multi-label-image-classification-model-python/)
 * [Using Python to Read Multiple JSON Files and Export Values to a CSV](https://blr.design/blog/python-multiple-json-to-csv/)
 * [How to flatten a pandas dataframe with some columns as json?](https://stackoverflow.com/questions/39899005/how-to-flatten-a-pandas-dataframe-with-some-columns-as-json)
 * [Iterating over a column and replacing a value with an extracted string [Pandas]](https://stackoverflow.com/questions/42741453/iterating-over-a-column-and-replacing-a-value-with-an-extracted-string-pandas)
 * [세번째 딥러닝 - 아이리스 품종 분류](https://opentutorials.org/course/4570/28987)
