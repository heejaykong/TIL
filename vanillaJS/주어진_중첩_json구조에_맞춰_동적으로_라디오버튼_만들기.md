# 주어진 중첩 json구조에 맞춰 동적으로 라디오버튼 만들기

**2022. 01. 27. 기록**

새로운 데이터 라벨링 기능을 만들다가 배운 점들을 기록한다.

대분류, 중분류, 소분류 ... 로 나뉘는 구조를 가진, 라벨구조 데이터에 맞춰 라디오버튼을 동적으로 생성하고 싶었다.

그리고 라디오 버튼을 선택하면, 해당 라디오버튼의 하위 라디오버튼들이 보이는 식의, 동적인 인터렉션도 만들고 싶었다.

마치 파일 디렉토리 구조처럼 말이다.

예를 들어 음식 이미지를 라벨링하는 상황이라고 치면, 다음과 같은 구조의 데이터가 주어질 수 있겠다.

```
food1  /* 대분류 */
  ㄴ food1-1  /* 중분류 */
      ㄴ food1-1-1  /* 소분류 */
      ㄴ food1-1-2
  ㄴ food1-2
      ㄴ food1-2-1
      ㄴ food1-2-2
      ㄴ food1-2-3      
food2
  ㄴ food2-1
      ㄴ food2-1-1
  ㄴ food2-2
      ㄴ food2-2-1
      ㄴ food2-2-2
      ㄴ food2-2-3
      ㄴ food2-2-4
```


그리고 위와 똑같은 모양으로 라디오버튼을, 앞서 말한 동적인 인터렉션과 함께 구현했다고 치면 다음과 같은 모습일 것이다.

![result](https://user-images.githubusercontent.com/18097984/151311758-140f0ef3-486c-4d2b-82d8-ef4ac696dd0f.gif)

이처럼 사용자가 라디오버튼을 클릭해서 라벨링할 수 있게 만들고, 나아가 라벨링한 정보를 post할 때 예외처리까지 해보는 과정을 이번 글에 적고자 한다.

---
할일은 다음과 같다.
1. json 양식 정의해보기
2. 주어진 json 형태대로 라디오버튼 생성하기 + 클릭할 때마다 하위 묶음 보여주기
3. 사용자가 라벨링 완료 후 post할 때 예외처리 로직 구현하기
    - 라벨을 최종 depth까지 다 선택하지 않은 경우:
        - 예) 총 depth가 2이면 `food1`, `food1-1`, `food1-1-1`를 전부 선택해야 하는데, `food1`, `food1-1`까지만 선택한 경우
    - 선택한 라벨들이 서로 직속 부모-자식 관계가 아닌 경우:
        - 예) `food2`, `food1-1`, `food1-1-2`
4. 기존 어노테이션 내역 로드하기

그럼 차례대로 살펴보자.

---
## 1. json 양식 정의해보기

앞선 소개에서 언급한 것과 같은 데이터의 구조를 재현하기 위해 json 포맷을 다음과 같이 설계했다.

각 분류마다 `depth`(0부터 시작)를 부여하고, 라벨명은 `class-name`에, 그리고 해당 라벨 하위에 또 라벨 그룹이 있다면 배열로 `class-map`에 넣었다.

_라벨구조 데이터 예시_
```json
[
    {
        "depth": "0",
        "class-name": "food1",
        "class-map": [
            {
                "depth": "1",
                "class-name": "food1-1",
                "class-map": [
                    {
                        "depth": "2",
                        "class-name": "food1-1-1"
                    },
                    {
                        "depth": "2",
                        "class-name": "food1-1-2"
                    }
                ]
            },
            {
                "depth": "1",
                "class-name": "food1-2",
                "class-map": [
                    {
                        "depth": "2",
                        "class-name": "food1-2-1"
                    },
                    {
                        "depth": "2",
                        "class-name": "food1-2-2"
                    },
                    {
                        "depth": "2",
                        "class-name": "food1-2-3"
                    }
                ]
            }
        ]
    },
    {
        "depth": "0",
        "class-name": "food2",
        "class-map": [
            {
                "depth": "1",
                "class-name": "food2-1",
                "class-map": [
                    {
                        "depth": "2",
                        "class-name": "food2-1-1"
                    }
                ]
            },
            {
                "depth": "1",
                "class-name": "food2-2",
                "class-map": [
                    {
                        "depth": "2",
                        "class-name": "food2-2-1"
                    },
                    {
                        "depth": "2",
                        "class-name": "food2-2-2"
                    },
                    {
                        "depth": "2",
                        "class-name": "food2-2-3"
                    },
                    {
                        "depth": "2",
                        "class-name": "food2-2-4"
                    }
                ]
            }
        ]
    }
]
```

이제부터 이 주어진 라벨구조 데이터를 **커스텀라벨** 이라고 칭하겠다.

그럼 이제 이 모양대로 프론트단에서 라디오버튼을 생성해보자.

## 2. 주어진 json 형태대로 라디오버튼 생성하기 + 클릭할 때마다 하위 묶음 보여주기

처음에 삽질을 정말 많이 했다. 구글링을 해도 내가 원하는 정보는 잘 안나왔다... 며칠을 붙잡았다. 힘들고 슬펐다

문제는 내 부족한 구글링 요령이었다. create radio button dynamically with nested json input <- 이런 키워드 조합 위주로 찾아봤다.

라디오버튼을 클릭하면 하위 묶음이 나타나는 로직이 담긴 [css코드 레퍼런스](https://stackoverflow.com/a/58487097)를 찾는 수확은 있었지만, 이걸 정확히 어떻게 내 상황에 적용해야 하는지 감이 전혀 잡히질 않았다.

내가 구현하려는 건 언제 어떤 depth의 커스텀라벨이 들어올 지 모르는, 동적인 라디오버튼 생성이었기 때문이었다.

상주해있던 개발자 오픈카카오톡 방에 질문을 올려봤다. 친절하신 분께서 js dynamically create unordered list <- 이런 키워드로 검색하셨고, 좀 더 내가 원하는 방향의 정보가 나오더라.

결정적인 레퍼런스는 [이곳](https://stackoverflow.com/a/56244873)에서 찾았다. ul 태그, li 태그, 그리고 재귀원리를 활용하는 것이 로직의 핵심이었다.

(사실 앞서 언급한 json 양식도, 처음부터 저렇게 설계되진 않았고 이 글을 좀 참고하며 구조를 바꿔보자고 내가 (json 양식을 처음 설계한 분에게) 제안했다.)

주어진 커스텀라벨(json)이 `CUSTOM_FORM`이라는 변수명에 담겨있다고 치자. 레퍼런스들을 참고하며 짠 전체 코드는 다음과 같다.

_html_
```html
...
<div class="radioBtnGroup"></div>
...
```
_javascript_
```javascript
    function buildLI(label){
        const $li = document.createElement('li');
        const $input = document.createElement('input');
        const $label = document.createElement('label');
        
        $input.setAttribute("type", "radio");
        $input.setAttribute("name", `depth${label.depth}`);
        $input.setAttribute("value", label["class-name"]);
        $input.setAttribute("id", label["class-name"]);

        $label.setAttribute("for", label["class-name"]);
        $label.innerHTML = `${label["class-name"]}`;

        $li.appendChild($input);
        $li.appendChild($label);
        if (label["class-map"]) $li.appendChild(buildUL(label["class-map"]));
        return $li;
    };

    function buildUL(customForm){
        const $ul = document.createElement('ul');
        customForm.forEach(label => {
            $ul.classList.add(`depth${label.depth}`)
            $ul.appendChild(buildLI(label));
        });
        return $ul;
    };

    document.querySelector('.radioBtnGroup').appendChild(
        buildUL(CUSTOM_FORM)
    );
```
_css_
```css
/* 최상위 라벨들(depth0) 말곤 전부 숨겨라 */
ul[class^="depth"]:not(.depth0) {
    display: none;
}

/* 체크한 라벨의 하위 라벨묶음을 보여라 */
:checked ~ ul[class^="depth"] {
    display: block;
}
```

여기까지 하면 주어진 커스텀라벨에 맞춰 라디오버튼이 생성되는 것과, 라벨을 하나 클릭할 경우 그 라벨의 하위 라벨묶음이 짠하고 나타나는 인터렉션까지 구현이 완성된다.

이제 예외처리를 해보자.

## 3. 사용자가 라벨링 완료 후 post할 때 예외처리 로직 구현하기

생각해본, 예외처리 해야 할 상황들은 다음과 같았다.

- 라벨을 최종 depth까지 다 선택하지 않은 경우:
    - 예) 총 depth가 2이면 `food1`, `food1-1`, `food1-1-1`를 전부 선택해야 하는데, `food1`, `food1-1`까지만 선택한 경우
- 선택한 라벨들이 서로 직속 부모-자식 관계가 아닌 경우:
    - 예) `food2`, `food1-1`, `food1-1-2`

전체 코드는 다음과 같다. 자세한 설명은 주석으로 적어놓았다.

```javascript
const ERROR_MSG = 
'마지막 라벨까지 선택되지 않거나, 선택된 라벨들이 서로 연결되지 않습니다. 작업을 다시 진행해 주세요.';
const radioBtnGroup = document.querySelector(".radioBtnGroup")

// 주어진 커스텀라벨(json)의 depth를 파악하는 함수다.
// 재귀로 구현했다.
function getDepthOfCustomForm(classMap, depth = 0){
    if (!classMap[0]['class-map']) {
        return depth
    }
    return getDepthOfCustomForm(classMap[0]['class-map'], depth + 1)
}

// 예외처리 1: 라벨을 최종 depth까지 다 선택하지 않은 경우
// 주어진 커스텀라벨의 depth를 파악하고, 선택한 라디오버튼의 갯수가 그 depth와 일치하는지 확인하는 로직이다.
function isLabelingIncomplete(){
    const selectedRadioBtns = radioBtnGroup.querySelectorAll("input:checked");
    const depthOfCustomForm = getDepthOfCustomForm(CUSTOM_FORM) + 1;

    if (selectedRadioBtns.length !== depthOfCustomForm) {
        console.log(selectedRadioBtns.length, depthOfCustomForm)
        return true
    }
    return false
}

// 예외처리 2: 선택한 라벨들이 서로 직속 부모-자식 관계가 아닌 경우
// 재귀로 구현했다. currentlyselectedLabels에는 현재의 어노테이션을 depth순으로 담은 배열이 주어진다. (예: ['food1', 'food1-1', 'food1-1-1'])
function isAnyLabelNotDirectlyRelated(classMap, currentlyselectedLabels, depth = 0) {
    if (!classMap){
        return false
    }
    for (let label of classMap){
        if (label['class-name'] === currentlyselectedLabels[depth]){
            return isAnyLabelNotDirectlyRelated(label['class-map'], currentlyselectedLabels, depth + 1)
        }
    }
    return true
}

// 아래 getCurrentlySelectedRadioBtns 함수는 현재의 어노테이션을 각각 객체로 담은 배열을 리턴한다.
// 예:
// [
//     {
//         "depth": "0",
//         "class-name": "food2"
//     },
//     {
//         "depth": "1",
//         "class-name": "food2-1"
//     },
//     {
//         "depth": "2",
//         "class-name": "food2-1-1"
//     }
// ]
function getCurrentlySelectedRadioBtns(){
    const selectedRadioBtns = radioBtnGroup.querySelectorAll("input:checked");
    let labels = []
    selectedRadioBtns.forEach(input => {
        const newObj = {
            "depth": input.name.replace('depth',''),  // 숫자만 저장
            "class-name": input.value
        }
        labels.push(newObj)
    });
    return JSON.stringify(labels)
}

// 현재의 어노테이션을 depth순으로 담은 배열을 리턴한다. (예: ['food1', 'food1-1', 'food1-1-1'])
function getCurrentlySelectedLabels(){
    const currentlySelectedRadioBtns = JSON.parse(getCurrentlySelectedRadioBtns())
    const currentlyselectedLabels = currentlySelectedRadioBtns.map(label => {
        return label['class-name']
    });
    return currentlyselectedLabels
}

// 현재 어노테이션 내역이 비정상인지, 2가지 경우를 확인하며 판단
function isAnnotationAbnormal(){
    const currentlySelectedLabels = getCurrentlySelectedLabels()
    return (
        isLabelingIncomplete() ||
        isAnyLabelNotDirectlyRelated(CUSTOM_FORM, currentlySelectedLabels)
    )
}

// post하는 버튼 클릭 시 예시
$("#post").click(function (){
    if (isAnnotationAbnormal()){  // 예외처리 함수
        alert(ERROR_MSG);
    } else {
        $.ajax({
            type: "POST",
            dataType: 'json',
            data: {
                'csrfmiddlewaretoken': CSRF_TOKEN,
                // ...
                'annodationResult': getCurrentlySelectedRadioBtns()  // 여기에 어노테이션 내역 담김
            },
            success: function (data) {
            // ...
            }
        })
    }
});
```

이렇게 post와 예외처리를 구현하고 나면, 앞서 언급한 것처럼 다음과 같은 시나리오들에 대응할 수 있게 된다.

- 라벨을 최종 depth까지 다 선택하지 않은 경우. 즉, 끝까지 다 선택하지 않은 경우.
    - 예1) 총 depth가 2이면 `food1`, `food1-1`, `food1-1-1`를 전부 선택해야 하는데, `food1`만 선택한 경우
    - 예2) 총 depth가 2이면 `food1`, `food1-2`, `food1-2-2`를 전부 선택해야 하는데, `food1`, `food1-2`까지만 선택한 경우
- 선택한 라벨들이 서로 직속 부모-자식 관계가 아닌 경우. 즉, 이어지지 않는 엉뚱한 라벨을 선택한 경우.
    - 예1) `food1`, `food2-1`, `food2-1-1`를 선택한 경우
    - 예2) `food2`, `food1-1`, `food1-1-1`를 선택한 경우 등...

위와 예시들과 같이 비정상적인 어노테이션 후 post버튼을 누르면, 다음과 같은 에러메시지를 alert으로 띄운다.

`"마지막 라벨까지 선택되지 않거나, 선택된 라벨들이 서로 연결되지 않습니다. 작업을 다시 진행해 주세요."`

그럼 post와 예외처리를 다 구현했으니 다음 단계로 넘어가보자.

## 4. 기존 어노테이션 내역 로드하기

이제 기존 라벨링한 내역을 확인하거나 수정하는 화면에서 쓰일 코드를 살펴보겠다.

기존 어노테이션 내역을 가져와 그대로 라디오버튼에 그려지도록 반영하는 코드는 아래처럼 간단하다.

```javascript
// 기존 어노테이션 내역 로드
function loadExistingAnnotationValue() {
    const savedResults = JSON.parse("get해온 기존 어노테이션 내역")
    savedResults.forEach(label => {
        // 기존 어노테이션 내역에 있는 라벨의 class-name과 동일한 라디오버튼을 찾고,
        // 그 라디오버튼의 checked 속성을 true로 만들기
        const matchingLabel = radioBtnGroup.querySelector(`ul input[id=${label["class-name"]}]`)
        matchingLabel.checked = true;
    });
}
loadExistingAnnotationValue();
```

## 마무리

동작은 해도, 완벽하게 최적화된 구현이라고 생각하지는 않는다. 우선 구현한 과정을 기록하는 데 의미를 두며 작성했다.

추후 아는 게 더 많아져 수정사항이 발생하면 계속 수정해나가려고 한다.

처음 구현해보는 기능이라 순탄치 않았으나 결국 구현에 성공해 뿌듯하다. 이 포스트를 나중에 읽으며 '별것도 아닌 것 가지고 고생했었구나'하고 생각하는 날이 왔으면 좋겠다.

### 참고
_html & css_
* [Nested Radio elements are treated like they are all on the same level on the DOM - jenryb의 답변](https://stackoverflow.com/a/58487097)
* [jenryb의 답변에 첨부된 jsfiddle 예시](http://jsfiddle.net/15y2tb0L/2/)
* [Is there a CSS selector by class prefix?](https://stackoverflow.com/questions/3338680/is-there-a-css-selector-by-class-prefix)
* [[MDN] CSS - :not()](https://developer.mozilla.org/ko/docs/Web/CSS/:not)

_js_
* [Creating nested list items dynamically - Bibberty의 답변](https://stackoverflow.com/a/56244873)
* [How to find the maximum "depth" of a python dictionary or JSON object?](https://stackoverflow.com/questions/30928331/how-to-find-the-maximum-depth-of-a-python-dictionary-or-json-object)
* [Javascript recursion, foreach loop won't exit? - Anthony C의 답변](https://stackoverflow.com/a/50025649)
