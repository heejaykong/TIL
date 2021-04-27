# RecyclerView 구현 및 Activity에 데이터 전달

**2020/01/03 기록**

몰입캠프 첫째주 과제(안드로이드 앱 만들기) 수행 중, 전화번호부와 사진첩을 구현하는 데 RecyclerView를 활용하고 싶었다.

많은 데이터를 한번에 보여주려고 하면 메모리 등 자원 문제로 버벅이거나 OOM(Out of Memory)이 발생할 수 있는데, 이런 성능문제를 해결하고 데이터를 효율적으로 보여주는 View가 RecyclerView다.

암튼 RecyclerView 자체를 구현하고, 해당 RecyclerView를 눌렀을 때 다른 화면으로 넘어가며 해당 데이터(전화번호 text 또는 이미지)도 넘겨주는 작업을 구현하고 싶었는데

아래 자료들을 참고하며 View, Activity, 리스너, Intent 등의 개념을 익힐 수 있었다.


### 참고
* [Android - Kotlin으로 RecyclerView를 구현하는 방법](https://codechacha.com/ko/android-recyclerview/)
* [How to pass an image from one activity to another activity in Kotlin?](https://www.tutorialspoint.com/how-to-pass-an-image-from-one-activity-to-another-activity-in-kotlin)
* [RecyclerView OnClickListener to New Activity](https://youtu.be/ZXoGG2XTjzU)
* [[Android][Kotlin] PutExtra 데이터 전달](https://blog.yena.io/studynote/2017/11/28/Android-Kotlin-putExtra.html)
* [How to pass drawable between activities in android?](https://www.tutorialspoint.com/how-to-pass-drawable-between-activities-in-android)
