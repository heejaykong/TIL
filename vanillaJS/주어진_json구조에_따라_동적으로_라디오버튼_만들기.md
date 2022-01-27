# 주어진 json구조에 맞춰 동적으로 라디오버튼 만들기

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
2. 위와 같이 주어진 json 형태대로 라디오버튼 생성하기 + 클릭할 때마다 동적으로 
3. 사용자가 라벨링 완료 후 post할 때 예외처리 로직 구현하기
    - 라벨을 최종 depth까지 다 선택하지 않은 경우:
        - 예) 총 depth가 2이면 `food1`, `food1-1`, `food1-1-1`를 전부 선택해야 하는데, `food1`, `food1-1`까지만 선택한 경우
    - 선택한 라벨들이 서로 직속 부모-자식 관계가 아닌 경우:
        - 예) `food2`, `food1-1`, `food1-1-2`

그럼 차례대로 살펴보자.

---
### 1. json 양식 정의해보기

앞선 소개에서 언급한 것과 같은 데이터의 구조를 재현하기 위해 json 포맷을 다음과 같이 설계했다.

각 분류마다 `depth`(0부터 시작)를 부여하고, 라벨명은 `class-name`에, 그리고 해당 라벨 하위에 또 라벨 그룹이 있다면 배열로 `class-map`에 넣었다.

_라벨 구조 데이터 예시_
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

그럼 이제 이 모양대로 프론트단에서 라디오버튼을 생성해보자.

### 2. 주어진 json 형태대로 라디오버튼 생성하기

처음에 삽질을 정말 많이 했다. 구글링을 해도 내가 원하는 정보는 잘 안나왔다... 며칠을 붙잡았다. 힘들고 슬펐다

문제는 내 부족한 구글링 요령이었다. create radio button dynamically with nested json input <- 이런 키워드 조합 위주로 찾아봤다.

라디오버튼을 클릭하면 하위 묶음이 나타나는 로직이 담긴 [css코드 레퍼런스](https://stackoverflow.com/a/58487097)를 찾는 수확은 있었지만, 이걸 정확히 어떻게 내 상황에 적용해야 하는지 감이 전혀 잡히질 않았다.

내가 구현하려는 건 언제 어떤 depth의 라벨구조 데이터가 들어올 지 모르는, 동적인 라디오버튼 생성이었기 때문이었다.

상주해있던 오픈카카오톡 방에 질문을 올려봤다. 친절하신 분께서 js dynamically create unordered list <- 이런 키워드로 검색하셨고, 좀 더 내가 원하는 방향의 정보가 나오더라.

결정적인 레퍼런스는 [이곳](https://stackoverflow.com/a/56244873)에서 찾았다. ul 태그, li 태그, 그리고 재귀원리를 활용하는 것이 로직의 핵심이었다.

(사실 json 양식 설계했던 것도, 처음부터 저렇게 설계하진 않았고 이 글을 좀 참고하며 구조를 바꿨다.)

주어진 라벨구조 데이터(json)이 `CUSTOM_FORM`이라는 변수명에 담겨있다고 치자. 레퍼런스들을 참고하며 짠 전체 코드는 다음과 같다.

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
        const $span = document.createElement('span');
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

여기까지 하면 주어진 라벨구조 데이터에 맞춰 라디오버튼이 생성되는 것과, 클릭할 경우 하위 라벨묶음이 보였다 안보였다 하는 인터렉션까지 구현이 완성된다.

이제 예외처리를 해보자.

### 3. 사용자가 라벨링 완료 후 post할 때 예외처리 로직 구현하기

생각해본, 예외처리 해야 할 상황들은 다음과 같았다.

- 라벨을 최종 depth까지 다 선택하지 않은 경우:
    - 예) 총 depth가 2이면 `food1`, `food1-1`, `food1-1-1`를 전부 선택해야 하는데, `food1`, `food1-1`까지만 선택한 경우
- 선택한 라벨들이 서로 직속 부모-자식 관계가 아닌 경우:
    - 예) `food2`, `food1-1`, `food1-1-2`

전체 코드는 다음과 같다.

마저 작성할 예정


```javascript
const ERROR_MSG = 
'마지막 라벨까지 선택되지 않거나, 선택된 라벨들이 서로 연결되지 않습니다. 작업을 다시 진행해 주세요.';
const radioBtnGroup = document.querySelector(".radioBtnGroup")

function getDepthOfCustomForm(classMap, depth = 0){
    if (!classMap[0]['class-map']) {
        return depth
    }
    return getDepthOfCustomForm(classMap[0]['class-map'], depth + 1)
}
// 예외처리 1
function isLabelingIncomplete(){
    const selectedRadioBtns = radioBtnGroup.querySelectorAll("input:checked");
    const depthOfCustomForm = getDepthOfCustomForm(CUSTOM_FORM) + 1;

    if (selectedRadioBtns.length !== depthOfCustomForm) {
        console.log(selectedRadioBtns.length, depthOfCustomForm)
        return true
    }
    return false
}
// 예외처리 2
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
function getCurrentlySelectedLabels(){
    const currentlySelectedRadioBtns = JSON.parse(getCurrentlySelectedRadioBtns())
    const currentlyselectedLabels = currentlySelectedRadioBtns.map(label => {
        return label['class-name']
    });
    return currentlyselectedLabels
}
function isAnnotationAbnormal(){
    const currentlySelectedLabels = getCurrentlySelectedLabels()
    return (
        isLabelingIncomplete() ||
        isAnyLabelNotDirectlyRelated(CUSTOM_FORM, currentlySelectedLabels)
    )
}

// 다음 버튼 클릭 시
$("#btnNext").click(function (){
    if (isAnnotationAbnormal()){
        alert(ERROR_MSG);
    } else {
        $.ajax({
            type: "POST",
            dataType: 'json',
            data: {
                'csrfmiddlewaretoken': CSRF_TOKEN,
                'type': 'final',
                'review': handleCheckClick(),
                'lines': getCurrentlySelectedRadioBtns()
            },
            success: function (data) {
                window.location = data.url;
            }
        })
    }
});
// 나가기 버튼 클릭 시
$('#btnOut').click(function() {
    if (isAnnotationAbnormal()){
        alert(ERROR_MSG);
    } else {
        $.ajax({
            type: "POST",
            dataType: 'json',
            data: {
                'csrfmiddlewaretoken': CSRF_TOKEN,
                'type': 'out',
                'review': handleCheckClick(),
                'lines': getCurrentlySelectedRadioBtns()
            },
            success: function (data) {
                window.location = data.url;
            }
        })
    }
});
// 임시저장 버튼 클릭 시
$('#btnTemp').click(function() {
    if (isAnnotationAbnormal()){
        alert(ERROR_MSG);
    } else {
        $.ajax({
            type: "POST",
            dataType: 'json',
            data: {
                'csrfmiddlewaretoken': CSRF_TOKEN,
                'type': 'temp',
                'review': handleCheckClick(),
                'lines': getCurrentlySelectedRadioBtns()
            },
            success: function (data) {
                window.location = data.url;
            }
        })
    }
});
// 이전 버튼 클릭 시
$('#btnPrev').click(function() {
    if (isAnnotationAbnormal()){
        alert(ERROR_MSG);
    } else {
        $.ajax({
            type: "POST",
            dataType: 'json',
            data: {
                'csrfmiddlewaretoken': CSRF_TOKEN,
                'type': 'prev',
                'review': handleCheckClick(),
                'lines': getCurrentlySelectedRadioBtns()
            },
            success: function (data) {
                window.location = data.url;
            }
        })
    }
});

// 기존 어노테이션 내역 로드
function loadExistingAnnotationValue() {
    const savedResults = JSON.parse(checked_labels.replace(/&quot;/g,'"'))
    savedResults.forEach(label => {
        // 기존 어노테이션 내역에 있는 라벨의 class-name과 동일한 라디오버튼을 찾고,
        // 그 라디오버튼의 checked 속성을 true로 만들기
        const matchingLabel = radioBtnGroup.querySelector(`ul input[id=${label["class-name"]}]`)
        matchingLabel.checked = true;
    });
}
loadExistingAnnotationValue();
```

### 참고
css
* [Nested Radio elements are treated like they are all on the same level on the DOM - jenryb의 답변](https://stackoverflow.com/a/58487097)
* [jenryb의 답변에 첨부된 jsfiddle 예시](http://jsfiddle.net/15y2tb0L/2/)

js
* [Creating nested list items dynamically - Bibberty의 답변](https://stackoverflow.com/a/56244873)
