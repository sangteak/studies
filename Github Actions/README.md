# 깃허브 액션

깃허브 액션은 엔드투엔드 깃허브 중심 소프트웨어개발 생명주기(SDLC: software development life cycle) 프로세스로, 자동화 플랫폼과 프레임워크를 제공합니다. 자동화 플랫폼은 깃허브 이벤트에 연결된 자동화 워크플로를 만들고 실행하는 수단이며, 프레임워크를 제공하여 복잡한 자동화 플랫폼을 체계적으로 구동하도록 돕습니다.

## 작동 원리
- 특정 이벤트가 깃허브 저장소에서 발생 합니다.
    > 이벤트는 고유한 SHA1 값과 그에 연동해 암호화된 깃 참조 값을 가집니다. 
    > - Git 참조 값: 어떤 브랜치인지 등 이벤트가 일어난 곳을 특정하는 참조 주소
- 워크플로(Workflow) 파일 전용 디렉토리(.github/workflows)를 검색해 특정 이벤트에 대응하게 만들어진 워크플로 파일을 찾습니다.
    > 이때 이벤트에 추가적인 퀄리파이어가 붙어져 있어 특정 조건에만 발동하게 하는 경우가 존재합니다. 예를 들어 특정 브랜치에서 push가 일어났을 경우로 제한하는 등의 것을 뜻합니다.
- 워크플로 파일 전용 디렉토리에서 대응하는 파일을 찾으면, 그 워크플로를 실행합니다.

## Workflow 스크립팅

### on 키워드
Workflow가 이벤트에 발동할 시점을 정하는 방법은 여러가지가 있으며, 이를 위해 `on 키워드`를 사용합니다. 다음은 `on 키워드`의 대표적인 사용 사례를 보여줍니다.

**이벤트 한개에 반응하는 Workflow**
```yml
on: push
```
**여러 이벤트에 반응하는 Workflow**
```yml
on: [push, pull_request]
```
**자세한 조건을 만족하는 이벤트에만 반응하는 Workflow**
```yml
on:
    push:
        branches:
            - main
            - 'rel/v*'
        tags:
            - 1.*
            - beta
        paths:
            - '**.ts'
```
**설정한 시각이나 주기에 따라 반응하는 Workflow**
```yml
on:
    scheduled:
        - cron: '30 5,15 * * *'
```
**특정한 수동 이벤트에 반응하는 Workflow**
```yml
on: [workflow-dispatch, repository-dispatch]
```
**다른 Workflow에서 호출한 Workflow
```yml
on: workflow_call
```
**깃허브에서의 일상적인 활동에 반응하는 Workflow**
```yml
on: issue_comment
```

### Step

스텝(Step)은 깃허브 액션의 최소 실행 단위 입니다. 스텝은 다음과 같이 구성될 수 있습니다.
- 사전에 정의된 액셜(함수, 프로시저 등)을 호출
- 러너가 실행할 셸 명령어의 조합

스텝은 `steps` 키워드로 시작하며, 사전에 정의된 액션은 `uses` 절을 통해 가져오며, 모든 셸 명령어는 `run` 절에 의해 실행됩니다. 

다음은 코드를 체크아웃하고, 특정 버전의 Go 환경을 설정한 다음, 이 Go 프로세스가 소스 파일을 실행합니다.
```yml
steps:
    # 사전 정의된 액션을 통해 코드 체크아웃
    - uses: actions/checkout@v3

    # go version 설치
    - name: setup Go version
        # 사전 정의된 액션을 통해 go 설치
        uses: action/setup-go@v2

        # 설치 시 Go 버전은 1.14.0으로 지정
        with:
            go-version: '1.14.0'

    # helloworld.go 파일 실행
    - run: go run helloworld.go
```

### Runner
러너(Runner)는 Workflow로 코드가 실행되는 물리/가상 서버 및 컨테이너를 말합니다. 

깃허브가 통제하는 환경에서 실행되는 자체적으로 제공하는 시스템을 러너로 사용할 수도 있고, 직접 설정, 호스팅하고 통제하는 인스턴스를 러너로 사용 할 수도 있습니다. 러너를 실행하면 서버가 깃허브 프레임워크와 상호작요하게끔 설정이 됩니다. 

이 시스템이 깃허브에 접속해서 Workflow와 사전 정의된 액션에 접근하고, 스텝을 실행하고, 결과를 보고하는 게 가능해지는 것을 의미합니다.

### Jobs
잡(Jobs)는 스텝을 모아두고 또 어떤 러너에서 실행할지 정의하는 컴포넌트 입니다. 잡은 보통 전체 Workflow의 중하위 목표를 완수하게끔 설계됩니다. 예를 들어 CI/CD 파이프라인의 경우 빌드, 테스트, 패키징을 각자 하나의 잡으로 설정해 총 3개의 잡으로 나눌 수 있습니다.

잡(Jobs)는 러너를 정의하는 부분을 제외하면, 프로그래밍 언어에서의 함수나 프로시저와 유사하다고 할 수 있습니다. 그 이유는 잡 내부는 실행할 명령어 라인과 사전 정의된 액션의 호출로 구성되기 때문입니다. 이것은 프로그래밍 언어에서의 함수와 같이 중간 중간 다른 함수나 프로시저를 호출하는 부분을 연상할 수 있습니다.

다음은 잡을 작성하는 간단한 예시 입니다. `build`라는 잡 이름을 지정하였으며, 위 스텝에서 사용한 코드를 `build` 잡에 포함시켰습니다. 또한 Runner를 지정하여, `ubuntu-latest`에서 동작하도록 하였습니다.

```yml
jobs:
    # Job의 이름 지정(build)
    build:
        # 깃허브가 호스팅하는 표준 우분투 운영체제 이미지로 러너 설정
        runs-on: ubuntu-latest
        steps:
            # 사전 정의된 액션을 통해 코드 체크아웃
            - uses: actions/checkout@v3

            # go version 설치
            - name: setup Go version
                # 사전 정의된 액션을 통해 go 설치
                uses: action/setup-go@v2

                # 설치 시 Go 버전은 1.14.0으로 지정
                with:
                    go-version: '1.14.0'

            # helloworld.go 파일 실행
            - run: go run helloworld.go
```

### 워크플로우(Workflow)
워크 플로(workflow)는 파이프라인과 비슷합니다. 우선 어떤 종류의 이벤트(입력)을 받을지, 어떤 조건일 때 실행할지 정합니다. 앞에서 이벤트(`on` 키워드)에서 살펴본 내용입니다. 

워크플로우(Workflow)의 실행은 다음과 같은 단계로 이루어집니다.
- 이벤트와 조건이 모두 맞으면 워크플로우(Workflow)는 그에 걸맞은 잡들을 발동
- 워크플로(Workflow)가 자기 밑의잡을 실행
- 잡은 그 안에 속한 각 스텝을 실행

전체적인 흐름은 CI 프로세스와 비슷합니다. 다음은 앞에서 만든 Go 프로그램을 처리하는 간단한 워크플로의 예시입니다.
```yml
# 워크플로의 이름은 `Simple Go Build`로 지정
name: Simple Go Build

# `on` 키둬드를 통해 push 이벤트에 발동하도록 설정
on:
    # branches를 통해 main 브런치를 지정하였으므로, main 브런치에 push 이벤트가 발생한 시점에 동작하도록 설정합니다.
    push:
        branches:
            - main

jobs:
    # Job의 이름 지정(build)
    build:
        # 깃허브가 호스팅하는 표준 우분투 운영체제 이미지로 러너 설정
        runs-on: ubuntu-latest
        steps:
            # 사전 정의된 액션을 통해 코드 체크아웃
            # github.com/actions/checkout에 설정된 액션을 사용하게 됩니다.
            # 추가 파라메터가 존재하지 않으므로 이 워크플로가 돌아가는 현재 저장소에서 해당 해동을 수행하게 됩니다.
            - uses: actions/checkout@v3

            # go version 설치
            - name: setup Go version
                # 사전 정의된 액션을 통해 go 설치
                uses: action/setup-go@v2

                # 설치 시 Go 버전은 1.14.0으로 지정
                with:
                    go-version: '1.14.0'

            # helloworld.go 파일 실행(쉘명령 실행)
            - run: go run helloworld.go
```

마지막으로 정리해보자면 다음과 같습니다.
- `yml`형식으로된 워크플로우 파일은 깃허브 리포지터리의 `<저장소>/.github/workflows` 경로에 존재해야 한다.
- 깃허브 액션은 해당 위치에서 워크플로 파일을 찾는다.
- 설정된 이벤트 조건을 확인하고, 자동으로 수행한다.

## 액션(Actions)

액션은 매우 간단한 것부터 매우 복잡한 것까지 다양합니다. 간단하게는 작은 쉘 스크립트를 하나 실행하는 수준부터 복잡하게는 콘텐츠 유효성 검사, 빌드, 취약성 검사, 패키징 등과 같은 CI/CD 태스크를 처리하는 데 필요한 대규모 구현 코드, 테스트 케이스 및 워크플로 전부를 다 관리하는 수준까지 구현합니다. 앞서 예제로 사용된 스크립트의 체크아웃(`uses: actions/checkout@v3`) 또한 이러한 액션의 일부입니다.

### 워크플로와 액션
액션은 다른 애플리케이션으로치면 하나의 모듈이나 플러그인을 말하며, 워크플로는 그 모듈이나플러그인이 담긴 파이프라인이나 스크립트라고 생각 할 수 있습니다.

따라서 워크플로는 액션을 호출해 스텝 단위로 일을 하라고 명령하고, 액션은 그렇게 맡은 일을 하는 과정에서 CI/CD, 자동화, 유효성 검사 등을 담당하는 워크플로들을 필요에 따라 호출합니다.

### 액션과의 상호작용
리포지터리의 코드를 액션으로 사용하려면 깃허브 리포지터리에 액션 파일이 필요합니다. 이 파일은 액션 자체에 대한 메타데이터가 포함된 파일로 action.yml(또는 action.yaml) 입니다. 이 파일은 입력, 출력 등 액션에 필요한 구성을 지정할 수 있습니다. 이 파일의 형식은 다음과 같습니다.

- 기본 정보(이름, 작성자, 설명)
- 입력(inputs)
- 출력(outputs)
- 실행(runs)

다음은 `checkout` 액션의 일부이며, `action.yml`의 형식을 맞게 작성한 예시를 제공합니다.
```yml
name: 'Checkout'
description: 'Checkout a Git repository at a particular version'
inputs:
    repository:
        description: 'Repository name with owner. For example, actions/checkout'
        default: ${{ github.repository }}
    ref:
        description: >
            The branch, tag or SHA to checkout. When checking out the repository that triggered a workflow, this defaults to the reference or SHA for that event. Otherwise, uses the default branch
    
    token:
        description: >
            Personal access token (PAT) used to fetch the repository. The PAT is configured with the local git config, which enabled your scripts to run authenticated git commands. The post-job step removes the PAT.
```

실제 해당 액션은 `- uses: actions/checkout@v3` 사용 시 with 절과 함께 사용 가능합니다. 해당 내용은 `https://github.com/marketplace/actions/checkout` 깃헙 마켓 플레이스의 README.md 파일의 Usage에서 확인 가능합니다.

### 액션 사용법
워크플로에서 모든 uses 절은 깃허브 리포지터리 안에서 액션의 경로를 참조하는 데 다음과 같이 작성합니다.

```yml
uses: actions/checkout@v3
```

uses 절에 사용된 경로는 github.com 뒤에 오는 깃허브 리포지터리 기준의 상대 경로입니다. 버전 번호(@ 기호 뒤 부분)는 여러 가지 방법으로 표현될 수 있습니다. 표현 버전은 기본적으로 시멘틱 버전을 사용하며, @v[메이저 버전]을 지정하면 해당 메이저 버전의 가장 최신 버전을 가리키게 됩니다. 또한 @v[전체 버전]을 명시할 수도 있습니다.
```yml
uses: actions/checkout@v3.6.0
```

### 공개 액션과 마켓플레이스
[액션 마켓플레이스](https://github.com/actions)는 깃허브가 직접 운영하는 중요한 마켓플레이스로 크리에이터가 자신이 만든 액션을 다른 사람들과 공유하는 공식 리포지터리 입니다. 사용자는 필요한 액션을 액션 마켓플레이스를 통해 검색 후 사용 할 수 있습니다. 또한 자신이 제작한 액션을 올려 공유할 수도 있습니다.


## 자체 호스팅 러너(Runners) 설정
깃허브에서 제공하는 러너 대신 러너를 직접 호스팅할 수도 있습니다. 러너를 자체적으로 호스팅하며 워크플로의 실행 환경을 보다 쉽게 구성하고 제어할 수 있습니다. 구성과 시스템 리소스, 러너에서 사용할 소프트웨어를 직접 선택하고 맞춤 설정해 온프레미스나 클라우드의 물리적 시스템, 가상 머신 또는 컨테이너 등 광범위한 인프라에서 러너를 실행합니다. 

러너를 직접 호스팅할 경우 다음과 같은 장/단점을 갖습니다.
**장점**
- 설정이 자유롭다.
- 깃허브에서 제공하는 러너를 사용할 경우 깃허브 플랜에 따라 일정량의 무료 사용량 제공 후 종량제 요금을 부과되지만, 자체 호스팅 러너는 기본적으로 무료이다. 
- 깃허브에서 제공하는 러너는 기본적으로 가상 시스템이지만 자체 호스팅 러너는 가상 혹스 물리 시스템을 선택할 수 있다.

**단점**
- 자체 호스팅 러너를 실행하기 위한 가상 또는 물리 시스템을 직접 관리해야한다.
- 깃허브 호스팅 러너는 OS, 패키지, 액션 애플리케이션과 기타 도구를 전담 제공해주지만 자체 호스팅용 러너는 애플리케이션만 제공된다.

### Ubuntu 환경에 자체 호스팅 러너 설치하기
- Runner를 적용하기 위한 Github Repository로 이동한다.
- 상단에 `Settings` 메뉴로 이동한다.

    ![actions](images/custom_runner_actions.png)
- 왼쪽 메뉴 중 `Actions`의 `Runners`로 이동 후 페이지 상단에 위치한 `new self-hosted runner` 버튼을 클릭합니다.
    ![new_runner](images/custom_runner_new_runner.png)
- 다음과 같이 페이지가 출력되며, 러너스 이미지를 설치할 OS를 선택합니다. 여기서는 Ubuntu에 설치하므로 `Linux`를 선택하였습니다.
    ![description](images/custom_runner_add_new_info.png)
- 터미널로 이동 Download 코드 블럭의 명령을 이용해 러너 이미지를 다운로드 합니다.
    ```
    # Create a folder
    $ mkdir actions-runner && cd actions-runner
    # Download the latest runner package
    $ curl -o actions-runner-linux-x64-2.321.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.321.0/actions-runner-linux-x64-2.321.0.tar.gz
    # Optional: Validate the hash
    $ echo "ba46ba7ce3a4d7236b16fbe44419fb453bc08f866b24f04d549ec89f1722a29e  actions-runner-linux-x64-2.321.0.tar.gz" | shasum -a 256 -c
    # Extract the installer
    $ tar xzf ./actions-runner-linux-x64-2.321.0.tar.gz
    ```
- 이후 Configure의 코드 블록 내용을 실행합니다. config.sh를 실행합니다.
    ```
    # Create the runner and start the configuration experience
    $ ./config.sh --url https://github.com/sangteak/simple-go-proj --token ADNFD5BYIJCIDGQUL6RMGQDHJ2Y6C
    ```
    해당 명령을 실행하면 다음과 같은 결과를 얻을 수 있으며, 설정 중간 입력력이 필요한 부분은 Enter를 누르면 기본 값으로 설정되므로 모두 Enter로 넘어갑니다.
    ![config.sh](images/custom_runner_configure.png)
- 마지막으로 `run.sh`를 실행하여 러너를 실행합니다.
    ```
    # Last step, run it!
    $ ./run.sh
    ```
    다음과 같은 출력이 나왔다면 정상실행된 것입니다.
    ![custom_running](images/custom_runner_running.png)
- 마지막으로 정상적으로 실행된것이 맞는지 확인을 위해 Github 페이지에서 `Settings`에 `Actions`의 `Runners`로 이동합니다. 이동 후 다음과 같이 Runners 목록에 설치한 자체 호스팅 러너가 표시되고 상태가 `idel`이면 정살 실행된 상태입니다.
    ![ccustom_running_check](images/custom_runner_running_status_check.png)
- 이 후 워크플로(Workflow)에서 `runs-on`절을 다음과 같이 변경해줍니다.
    ```
    # Use this YAML in your workflow file for each job
    runs-on: self-hosted
    ```
- 이후 워크플로(Workflow)가 실행될 때 자체 호스팅 러너의 로그를 확인해보면 해당 러너를 통해 워크플로의 Job들이 실행되는 것을 확인할 수 있습니다.

## CI 스크립트 작성
해당 스크립트는 [Bytes](https://github.com/sangteak/bytes) 리파지토리의 push, pull_request 이벤트 발생 시 동작하는 스크립트 입니다. Bytes 라이브러리 빌드 환경은 해당 리파지토리의 `README.md` 내용을 참고하세요.

해당 리파지토리는 다음과 같은 디렉토리 구조를 갖습니다.

| 구분           | 설명                                                 |
| -------------- | ---------------------------------------------------- |
| Include/Bytes  | Bytes의 헤더 파일을 관리합니다.                      |
| Tests          | Bytes에 대한 테스트 코드를 관리합니다.               |
| Docs           | 프로젝트에 필요한 문서를 관리합니다.                 |
| Tool           | CLang-Format 등 프로젝트에 필요한 도구를 관리합니다. |
| ToolChain      | CMake 툴체인을 관리합니다.                           |
| CMAkeLists.txt | CMake 빌드를 위한 설정 파일 입니다.                  |
| conanfile.py   | conan 의존성 관리자를 위한 설정 파일입니다.          |

```yml
# 액션 이름 CI로 설정
name: CI

# main, features/* 브랜치 push 및 pull_request 시 발동하도록 이벤트 지정
on: 
  push:
    branches: [main, feature/*]
  pull_request:
    branches: [main, feature/*]

jobs:
  process:
    # 러너를 ubuntu-latest로 지정합니다.
    runs-on: ubuntu-latest

    # 스텝을 정의 합니다.
    steps:
      # 코드를 체크아웃 합니다.
      - name: Checkout
        uses: actions/checkout@v4

      # 빌드를 위한 도구를 설치합니다.
      - name: Install build-essentials
        if: success()
        run: |
          sudo apt update
          sudo apt-get install -y gcc cmake make python3-pip

      # conan 툴 설치 및, conan install 실행
      # - conan 설치
      # - profile 생성 및 C++17로 수정
      # - conan install 실행(Debug, Release)
      - name: Install conan2.x
        if: success()
        run: |
          sudo pip install conan
          rm -f ~/.conan2/profiles/default
          conan profile detect
          sed -i -E 's/gnu[0-9]{2}/gnu17/g' ~/.conan2/profiles/default
          conan install . --build=missing --settings=build_type=Debug
          conan install . --build=missing --settings=build_type=Release

      # Build 및 테스트 실행
      # - LD_LIBRARY_PATH 등록을 위해 conanrun.sh 실행(Debug, Release)
      #   source conanrun.sh를 실행할 경우 MacOS, Linux 환경에 따라 환경 변수를 설정해줍니다.
      # - configure preset을 'conan-debug'로 설정
      # - cmake를 통해 build preset을 `conan-debug`로 설정하여 빌드
      # - Tests 실행
      - name: Build&Tests
        if: success()
        run: |
          source ./build/Debug/generators/conanrun.sh
          source ./build/Release/generators/conanrun.sh
          cmake --preset conan-debug
          cmake --build --preset conan-debug
          ./build/Debug/Tests/Tests
```

## 참고
러닝 깃허브 액션(Learning GitHub Actions) - 한빛미디어