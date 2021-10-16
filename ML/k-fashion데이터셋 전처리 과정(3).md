**2021. 09. 05. 기록**

# K-Fashion 이미지 데이터셋을 전처리해 보자 (3) - Generate Train, Validation, Test 편

## 1. 배경설명

> **이전 포스트: [K-Fashion 이미지 데이터셋을 전처리해 보자 (2) - Data Cleaning 편](https://github.com/heejaykong/TIL/blob/main/ML/k-fashion%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%20%EC%A0%84%EC%B2%98%EB%A6%AC%20%EA%B3%BC%EC%A0%95(2).md#k-fashion-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EB%8D%B0%EC%9D%B4%ED%84%B0%EC%85%8B%EC%9D%84-%EC%A0%84%EC%B2%98%EB%A6%AC%ED%95%B4-%EB%B3%B4%EC%9E%90-2---data-cleaning-%ED%8E%B8)**

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

### 참고
* []()
* []()
