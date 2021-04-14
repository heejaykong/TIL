# import kotlinx 문에 오류 뜰 때

**2020/01/02 기록**

오류 뜨는 이유:

안드로이드 스튜디오가 4.1로 업데이트 되면서 기본 플러그인으로 제공하던 `kotlin-android-extensions`이 제거되고, 기본 `com.android.application`과 `kotlin-android`만 남게 되었다.

따라서 `kotlin-android-extensions` 플러그인을 사용하려면 아래처럼 직접 추가하면 되는데, 좋은 플러그인은 아니란다...

아무튼 방법

1. bulid.gradle에 들어가서

2. kotlin-android-extensions 플러그인 추가

3. 사진에 표시된 Sync Now 클릭


### 참고
* [import kotlinx.android.synthetic.main.activity_main.* 이 안될 경우 해결 방법](https://ku-hug.tistory.com/61)

* [Android Studio 4.1에서 제거된 Kotlin Android Extensions을 알아보자](https://thdev.tech/android/2020/10/07/Remove-kotlinx-synthetic/)
