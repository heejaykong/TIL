**2021.09.17. 기록**

openWeatherMap api를 리액트에서 사용하던 중,

useEffect의 dependency array 안에 날씨정보가 담긴 **객체**를 넣으니 useEffect가 무한대로 fire되는 현상을 목격할 수 있었다.

<img src="https://user-images.githubusercontent.com/18097984/133751533-fe98592c-ec1f-430b-a1f6-a70c130dc456.png" width="65%" alt="code"/>

![image](https://user-images.githubusercontent.com/18097984/133750103-3bc73010-c437-4919-b986-cb914fa1e7f5.png)

_useEffect가 계속 실행돼서 `console.log(response)`가 끊임없이 불러와지는 모습_

원인은, dependency array에 object를 넣으면 object의 reference가 변경될 때 마다 실행되기 때문이었다.

만일 해당 object 내부의 절대적 값이 변경될 때 마다 useEffect가 fire되기를 원한다면, 여러가지 방법이 있는데(참고링크 두번째),

나는 `use-deep-compare-effect`의 useDeepCompareEffect를 useEffect 대신에 사용하는 것으로 해결하기로 했다.

<img src="https://user-images.githubusercontent.com/18097984/133752528-b90d5b6b-b159-4d03-a82e-f282c76baec4.png" width="65%" alt="code"/>

_useDeepCompareEffect로 useEffect를 대체하니 더이상 해당 현상이 발생하지 않는다._

### 참고
* [React useEffect 의 dependency array](https://sgwanlee.medium.com/useeffect%EC%9D%98-dependency-array-ebd15f35403a)
* [Object & array dependencies in the React useEffect Hook](https://www.benmvp.com/blog/object-array-dependencies-react-useEffect-hook/)
