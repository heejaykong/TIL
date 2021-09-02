**2021. 09. 02. 기록**

# vscode에서 jupyter notebook 사용하기

## (1) 가상환경 설정

아나콘다를 다운받아 jupyter notebook으로 데이터 전처리 작업을 하다가

여러모로 더 편한 UI를 가진 vscode에서 사용해보고 싶어 .ipynb파일을 vscode에서 열어보았다.

`Ctrl` + `Shift` + `P`를 눌러 Python Interpreter를 셀렉해도 코드를 실행시키면 오류가 자꾸 뜨는 게, 가상환경이란 개념을 이해해야 해결할 수 있을 것 같았다.

가상환경이란 말그대로 가상의 작업환경을 의미하는데, dependencies 및 버전관리를 위해 독립적으로 작업환경을 마련할 수 있는 체계라고 이해했다.

내가 A라는 가상환경을 만들면, A라는 곳에 tensorflow, pytorch 등등의 외부 라이브러리를 설치하는 등 내 입맛대로 작업환경을 구축할 수 있고,

이렇게 만든 가상환경들을 마치 폴더처럼 관리해 나중에 재사용하거나 공유할 수도 있다.

사용하는 주요 명령어들

1. 가상환경 새로 만들고 싶을 때

`$ conda create -n 가상환경이름 python=버전`

2. 내가 만든 가상환경들을 한눈에 확인하고 싶을 때

`$ conda info --envs`

3. 가상환경으로 들어갈 때(또는 나갈 때)

`$ conda activate(또는 deactivate) 가상환경이름`

4. 특정 가상환경에 뭔갈 설치해야할 때(pandas라고 치면)

`$ conda install -n 가상환경이름 pandas`

5. 들어간 가상환경에 내가 뭐뭐 설치했는지 보고 싶을 때

`$ conda list`


## (2) 패키지 설치했는데도 모듈이 없다며 import가 안 될 때

분명 Anaconda prompt로 numpy를 원하는 가상환경에 설치하고, 그 가상환경에서 vscode로 작업을 하는데 `import numpy`문에서 자꾸 다음과 같은 오류가 발생했다.

`ImportError: DLL load failed: The file cannot be accessed by the system`

구글링 끝에 **관리자로 Anaconda prompt를 실행**하면 해결된다는 것을 알았다.

Anaconda prompt를 관리자로 실행해, 해당 가상환경으로 들어가 numpy를 지우고 다시 설치하니 vscode에서 코드가 잘 작동했다.

## (3) 위 방법이 안 될 때
> To ensure the environment is set up well from a shell perspective, one option is to use an Anaconda prompt with the activated environment to launch VS Code using the `code .` command. At that point you just need to select the interpreter using the Command Palette or by clicking on the status bar.

Anaconda prompt에서 작업하고자 하는 디렉토리로 들어가서 `code .` 로 vscode 실행시켜주고,

Python Interpreter를 다시 지정해주니까 작동했음ㅠ 휴

### 참고
* [[Anaconda] 아나콘다 가상환경의 개념 및 활용방법](https://yganalyst.github.io/pythonic/anaconda_env_1/)
* [[파이썬 에러 해결 Tip] ImportError: DLL load failed: 시스템에서 파일에 액세스할 수 없습니다.](https://3rdscholar.tistory.com/81)
* [Using Python environments in VS Code](https://code.visualstudio.com/docs/python/environments#_create-a-conda-environment)
