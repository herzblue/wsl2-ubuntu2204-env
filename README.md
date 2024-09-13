
# WSL2 Ubuntu 22.04.3 - 초기 설정 가이드
<summary>목차</summary>

- [WSL2 Ubuntu 22.04.3 - 초기 설정 가이드](#wsl2-ubuntu-22043---초기-설정-가이드)
  - [WSL2 Ubuntu 22.04.3에 zsh 설치 및 powerlevel10k 테마 설정 가이드](#wsl2-ubuntu-22043에-zsh-설치-및-powerlevel10k-테마-설정-가이드)
  - [WSL2 Ubuntu 22.04.3 환경에서 Poetry를 활용한 Python 설정 가이드](#wsl2-ubuntu-22043-환경에서-poetry를-활용한-python-설정-가이드)
  - [Poetry와 함께 다양한 Python 버전 관리하기: pyenv 활용 가이드 (3.8 최신, 3.11.9 버전 예시)](#poetry와-함께-다양한-python-버전-관리하기-pyenv-활용-가이드-38-최신-3119-버전-예시)
- [python 설치에 필요한 라이브러리 설치](#python-설치에-필요한-라이브러리-설치)
  - [Poetry와 함께 다양한 Python 버전 관리하기: pyenv \& update-alternatives 활용 가이드 (3.8 최신, 3.11.9 버전 예시)](#poetry와-함께-다양한-python-버전-관리하기-pyenv--update-alternatives-활용-가이드-38-최신-3119-버전-예시)
  - [WSL2에서 Windows Git Credential Manager 사용하기](#wsl2에서-windows-git-credential-manager-사용하기)
  - [Google Cloud 설정](#google-cloud-설정)
  - [Linux에 gcloud CLI 설치하기](#linux에-gcloud-cli-설치하기)
    - [1. Python 버전 확인](#1-python-버전-확인)
    - [2. gcloud CLI 다운로드](#2-gcloud-cli-다운로드)
    - [3. 압축 해제](#3-압축-해제)
    - [4. 설치 스크립트 실행](#4-설치-스크립트-실행)
    - [5. 터미널 재시작](#5-터미널-재시작)
    - [6. gcloud CLI 초기화](#6-gcloud-cli-초기화)
    - [7. 추가 구성 요소 설치 (선택 사항)](#7-추가-구성-요소-설치-선택-사항)
    - [설치 확인](#설치-확인)
  

## WSL2 Ubuntu 22.04.3에 zsh 설치 및 powerlevel10k 테마 설정 가이드

**1. zsh 설치:**

```bash
sudo apt update
sudo apt install zsh
```

**2. zsh를 기본 쉘로 설정:**

```bash
chsh -s $(which zsh)
```
- 터미널을 껐다 켜야 변경 사항이 적용됩니다.

**3. zsh가 기본 쉘로 설정되었는지 확인:**

```bash
su - $USER
echo $SHELL
```
- 출력 결과가 `/usr/bin/zsh` 이면 성공적으로 설정된 것입니다.

**4. oh-my-zsh 설치:**

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

**5. powerlevel10k 테마 설치:**

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

**6. powerlevel10k 테마 적용:**

1. `~/.zshrc` 파일을 편집합니다.
   ```bash
   vi ~/.zshrc
   ```

2. `ZSH_THEME`  변수 값을  `powerlevel10k/powerlevel10k`로 변경합니다.
   ```
   ZSH_THEME="powerlevel10k/powerlevel10k"
   ```

3. 파일을 저장하고 vi 편집기를 종료합니다 (`:wq`).

**7. 변경 사항 적용:**

```bash
source ~/.zshrc
```

**8. 폰트 설정 (선택 사항):**

- powerlevel10k 테마는 "Meslo Nerd Font"를 권장합니다. 
- Windows Terminal에서 설정 > 기본값 > 모양 > 글꼴에서 "MesloLGS NF" 폰트를 선택하세요.
- 다운로드 링크: [Meslo Nerd Font](https://github.com/romkatv/powerlevel10k/?tab=readme-ov-file#user-content-fonts)

**9. powerlevel10k 설정 마법사 실행 (선택 사항):**

- 터미널을 새로 열면 powerlevel10k 설정 마법사가 자동으로 실행됩니다. 
- 마법사를 통해 원하는 테마 스타일을 선택하고 세부 설정을 구성할 수 있습니다. 
- 설정 마법사를 다시 실행하려면 다음 명령어를 실행하세요.
  ```bash
  p10k configure
  ```

이제 WSL2 Ubuntu 22.04.3 환경에서 멋지게 꾸며진 zsh 터미널을 사용할 수 있습니다! 😊

<br/>
<br/>

## WSL2 Ubuntu 22.04.3 환경에서 Poetry를 활용한 Python 설정 가이드

Poetry는 Python 프로젝트의 의존성 관리와 패키징을 간편하게 해주는 도구입니다. WSL2 Ubuntu 22.04.3에서 Poetry를 사용하여 Python 프로젝트를 설정하는 방법을 단계별로 알려드리겠습니다.

**1. Poetry 설치**

Poetry는 공식적으로 Python 3.7 이상 버전을 지원합니다. Ubuntu 22.04.3는 기본적으로 Python 3.10이 설치되어 있으므로, 별도의 Python 버전 설치 없이 Poetry를 설치할 수 있습니다.

* **공식 설치 스크립트 사용:**

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

* **수동 설치 (선택 사항):** 
  - 공식 설치 스크립트가 제대로 작동하지 않을 경우,  [https://python-poetry.org/docs/#installation](https://python-poetry.org/docs/#installation) 에서 수동 설치 방법을 참고하세요.

**2. Poetry 환경 변수 설정**

Poetry를 설치한 후, PATH 환경 변수에 Poetry bin 디렉토리를 추가해야 합니다.  

* **현재 터미널 세션에만 적용:**
  ```bash
  export PATH="$HOME/.local/bin:$PATH"
  ```

* **영구적으로 적용:** 
  -  `~/.bashrc` 또는  `~/.zshrc` 파일 (사용 중인 쉘에 따라 다름) 끝에 다음 줄을 추가합니다.
    ```bash
    export PATH="$HOME/.local/bin:$PATH"
    ```
  - 변경 사항을 적용하려면 터미널을 껐다 켜거나 `source ~/.bashrc` (또는 `source ~/.zshrc`) 명령어를 실행합니다.

- 스크립트 실행 후, 아래 명령어로 Poetry가 정상적으로 설치되었는지 확인할 수 있습니다.
  ```bash
  poetry --version
  ```

**3. Python 버전 관리 (pyenv)**

**3.1 pyenv 설치**

Ubuntu 22.04.3에 pyenv를 설치하기 위해 다음 명령어를 실행합니다.

```bash
sudo apt update
sudo apt install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev curl git
```

```bash
curl https://pyenv.run | bash
```

**3.2 pyenv 환경 변수 설정**

pyenv를 설치한 후, 쉘 환경 설정 파일에 pyenv 관련 설정을 추가해야 합니다.

- **현재 터미널 세션에만 적용:**
  ```bash
  export PYENV_ROOT="$HOME/.pyenv"
  export PATH="$PYENV_ROOT/bin:$PATH"
  eval "$(pyenv init -)"
  eval "$(pyenv virtualenv-init -)"
  ```

- **영구적으로 적용:** 
  - `~/.bashrc` 또는  `~/.zshrc` 파일 (사용 중인 쉘에 따라 다름) 끝에 다음 줄을 추가합니다.
    ```bash
    export PYENV_ROOT="$HOME/.pyenv"
    export PATH="$PYENV_ROOT/bin:$PATH"
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
    ```
  - 변경 사항을 적용하려면 터미널을 껐다 켜거나 `source ~/.bashrc` (또는 `source ~/.zshrc`) 명령어를 실행합니다.

**3.3. Python 버전 설치**

pyenv를 사용하여 원하는 Python 버전을 설치합니다.  예를 들어, Python 3.8.10을 설치하려면 다음 명령어를 실행합니다.

```bash
pyenv install 3.8.10
```

- 설치 가능한 Python 버전 목록은  `pyenv install --list` 명령어를 사용하여 확인할 수 있습니다.

**3.4. Poetry 프로젝트에서 Python 버전 지정**

Poetry를 사용하여 새로운 프로젝트를 생성할 때, `pyproject.toml`  파일에 원하는 Python 버전을 지정할 수 있습니다. 

```bash
poetry new my-project --python 3.8.10
```

- 이 명령어는 "my-project" 디렉토리와  `pyproject.toml` 파일을 생성하고,  `python = "^3.8.10"`  설정을 추가하여 Python 3.8.10 버전을 사용하도록 지정합니다.

**3.5. 가상 환경 생성 및 활성화**

프로젝트 디렉토리로 이동하여 가상 환경을 생성하고 활성화합니다.

```bash
cd my-project
poetry shell
```

- Poetry는  `pyproject.toml`  파일에 지정된 Python 버전을 사용하여 가상 환경을 생성합니다. 


**4. 새로운 Python 프로젝트 생성**

Poetry를 사용하여 새로운 Python 프로젝트를 생성하는 것은 매우 간단합니다. 

```bash
poetry new my-project
```

- 이 명령어는 "my-project"라는 디렉토리를 생성하고, 그 안에 기본적인 프로젝트 구조와  `pyproject.toml` 파일을 생성합니다.

**5. 가상 환경 생성 및 활성화**

Poetry는 프로젝트마다 독립적인 가상 환경을 생성하여, 의존성 충돌을 방지합니다. 

```bash
cd my-project
poetry shell
```

- `poetry shell` 명령어는 프로젝트에 대한 가상 환경을 생성하고 활성화합니다.  
- 가상 환경이 활성화되면, 터미널 프롬프트 앞에 가상 환경 이름이 표시됩니다.

**6. 패키지 설치 및 관리**

Poetry를 사용하면  `pyproject.toml`  파일에 프로젝트 의존성을 정의하고 관리할 수 있습니다. 

* **새로운 패키지 설치:**
  ```bash
  poetry add requests
  ```
  - "requests" 패키지가 프로젝트에 설치되고,  `pyproject.toml`  파일에 의존성 정보가 자동으로 추가됩니다.

* **패키지 제거:**
  ```bash
  poetry remove requests
  ```

* **모든 의존성 설치:**
  ```bash
  poetry install
  ```
  - `pyproject.toml` 파일에 정의된 모든 의존성을 설치합니다.

**7. 프로젝트 실행**

가상 환경이 활성화된 상태에서, 일반적인 Python 명령어를 사용하여 프로젝트를 실행할 수 있습니다. 

```bash
python src/my_project/main.py
```

**추가 팁:**

* Poetry는 다양한 명령어와 옵션을 제공합니다.  `poetry --help`  명령어를 사용하여 사용 가능한 명령어 목록을 확인하거나,  [https://python-poetry.org/docs/](https://python-poetry.org/docs/) 에서 공식 문서를 참고하세요.
* VS Code와 Poetry를 함께 사용하면 더욱 편리한 개발 환경을 구축할 수 있습니다. "Python" 확장 프로그램을 설치하고, Poetry를 사용하여 생성한 가상 환경을 VS Code의 Python 인터프리터로 설정하면 됩니다.

이제 WSL2 Ubuntu 22.04.3 환경에서 Poetry를 사용하여 Python 프로젝트를 효율적으로 관리하고 개발할 수 있습니다! 😊 궁금한 점이 있다면 언제든지 질문해 주세요! 

<br/>

## Poetry와 함께 다양한 Python 버전 관리하기: pyenv 활용 가이드 (3.8 최신, 3.11.9 버전 예시)

Poetry는 훌륭한 Python 프로젝트 관리 도구이지만, Python 버전 자체를 관리하는 기능은 제공하지 않습니다. pyenv를 함께 사용하면 프로젝트별로 필요한 Python 버전을 쉽게 설치하고 전환하여 Poetry 환경을 유연하게 구성할 수 있습니다.

**1. pyenv 설치 및 설정**

- Ubuntu 22.04.3에 pyenv를 설치하는 방법은 이전 답변에서 설명한 것과 동일합니다. 
- 아직 설치하지 않았다면, 이전 답변의 "7. pyenv" 섹션을 참고하여 pyenv를 설치하고 환경 변수를 설정하세요.

**2. Python 버전 설치 (3.8 최신, 3.11.9)**

- pyenv를 사용하여 원하는 Python 버전을 설치할 수 있습니다.  `pyenv install --list`  명령어로 설치 가능한 Python 버전 목록을 확인하고,  `pyenv install <버전>`  명령어로 설치합니다.
- 예시:
    ```bash
   # python 설치에 필요한 라이브러리 설치
   sudo apt update
   sudo apt install libbz2-dev libsqlite3-dev liblzma-dev

    pyenv install 3.8.17 # 2024년 9월 현재 3.8 최신 버전
    pyenv install 3.11.9
    ```

**3. Poetry 프로젝트 생성 및 Python 버전 지정**

- 새로운 Poetry 프로젝트를 생성할 때,  `poetry new`  명령어에  `--python`  옵션을 사용하여 Python 버전을 지정할 수 있습니다.
- 예시: 
    - Python 3.8.17을 사용하는 프로젝트 생성:
        ```bash
        poetry new my-project-3.8 --python 3.8.17
        ```
    - Python 3.11.9를 사용하는 프로젝트 생성:
        ```bash
        poetry new my-project-3.11 --python 3.11.9
        ```
- 이렇게 하면 `pyproject.toml` 파일에 `python = "^3.8.17"` 또는 `python = "^3.11.9"` 설정이 자동으로 추가됩니다.

**4. 프로젝트별 Python 버전 설정**

- 기존 Poetry 프로젝트에서 Python 버전을 변경하려면  `pyproject.toml`  파일의  `python`  설정을 직접 수정합니다.
- 예시:  `my-project`  프로젝트의 Python 버전을 3.11.9으로 변경
    1.  `my-project`  디렉토리로 이동:  `cd my-project`
    2.  `pyproject.toml`  파일을 열고  `python`  설정을  `"^3.11.9"`으로 변경합니다.
    3.  가상 환경을 다시 생성합니다:  `poetry env remove python && poetry shell`  (기존 가상 환경을 제거하고 새 버전으로 다시 생성)

**5. 전역 Python 버전 설정 (선택 사항)**

- 특정 Python 버전을 시스템 전체에서 기본값으로 사용하려면  `pyenv global`  명령어를 사용합니다.
- 예시: Python 3.11.9를 전역 기본 버전으로 설정
    ```bash
    pyenv global 3.11.9
    ```

**6. Python 버전 확인**

- 현재 활성화된 Python 버전은  `python --version` 또는  `python3 --version`  명령어로 확인할 수 있습니다.
- pyenv가 관리하는 Python 버전 목록은  `pyenv versions`  명령어로 확인할 수 있습니다.

**7. Poetry와 pyenv 연동 팁**

- **가상 환경 자동 활성화:**  `pyenv virtualenv-init -`  설정을 쉘 환경 설정 파일에 추가하면, Poetry 프로젝트 디렉토리로 이동할 때 자동으로 해당 프로젝트의 가상 환경이 활성화됩니다.
- **VS Code 연동:**  VS Code의 "Python" 확장 프로그램을 사용하는 경우,  "Python: Select Interpreter"  명령을 사용하여 Poetry가 생성한 가상 환경을 선택할 수 있습니다.

**주의 사항:**

- 여러 Python 버전을 사용할 때는 각 버전의 의존성 패키지가 서로 호환되지 않을 수 있습니다.  프로젝트별로 가상 환경을 사용하여 의존성 충돌을 방지하는 것이 중요합니다.

pyenv를 활용하여 3.8 최신, 3.11.9 버전을 포함한 다양한 Python 버전을 효율적으로 관리하고, Poetry와 함께 편리하게 Python 프로젝트를 개발하세요! 😊 


<br/>
<br/>


## Poetry와 함께 다양한 Python 버전 관리하기: pyenv & update-alternatives 활용 가이드 (3.8 최신, 3.11.9 버전 예시)

Python 버전 관리에는 `pyenv` 뿐만 아니라 `update-alternatives` 도 활용할 수 있습니다.  `update-alternatives`는 시스템에 여러 버전의 프로그램이 설치되어 있을 때, 어떤 버전을 기본으로 사용할지 설정하는 데 사용되는 Ubuntu 시스템 도구입니다. `pyenv`가 Python 버전을 격리된 환경에 설치하고 관리하는데 중점을 둔다면,  `update-alternatives`는 시스템 레벨에서 Python 버전을 전환하는 데 유용합니다.

**1. pyenv 설치 및 Python 버전 설치**

- 1, 2 단계는 이전 답변과 동일합니다. `pyenv`를 설치하고 3.8 최신 버전 (3.8.17)과 3.11.9 버전을 설치합니다. 

**2. update-alternatives 설정**

- pyenv로 설치한 Python 버전들을 `update-alternatives`에 등록합니다. 

   ```bash
   sudo update-alternatives --install /usr/bin/python3 python3 /home/<사용자 이름>/.pyenv/versions/3.8.17/bin/python3 1
   sudo update-alternatives --install /usr/bin/python3 python3 /home/<사용자 이름>/.pyenv/versions/3.11.9/bin/python3 2

   # example
   sudo update-alternatives --install /usr/bin/python3 python3 /home/herzblue/.pyenv/versions/3.8.17/bin/python3 1
   sudo update-alternatives --install /usr/bin/python3 python3 /home/herzblue/.pyenv/versions/3.11.9/bin/python3 2
   ```
   - `<사용자 이름>`은 실제 사용자 이름으로 바꿔야 합니다.
   -  숫자 1과 2는 우선순위를 나타냅니다. 숫자가 높을수록 우선순위가 높습니다.

**3. 기본 Python 버전 선택**

- `update-alternatives`를 사용하여 기본 Python 3 버전을 선택합니다.

   ```bash
   sudo update-alternatives --config python3
   ```
   - 실행하면 설치된 Python 버전 목록이 나타납니다.  원하는 버전의 번호를 입력하고 Enter 키를 누릅니다.

- `auto mode` 설정

   ```bash
   sudo update-alternatives --auto <alternative name>
   ```

**4. Poetry 프로젝트 생성 및 Python 버전 지정**

- 새로운 Poetry 프로젝트를 생성할 때 `--python` 옵션으로 Python 버전을 지정합니다.

   ```bash
   # Python 3.8.17 프로젝트
   poetry new my-project-3.8 --python 3.8.17 
   # Python 3.11.9 프로젝트
   poetry new my-project-3.11 --python 3.11.9
   ```

**5. 가상 환경 생성 및 활성화**

- Poetry 프로젝트 디렉토리로 이동하여 가상 환경을 생성하고 활성화합니다.

   ```bash
   cd my-project-3.8  # 또는 my-project-3.11
   poetry shell
   ```

**6. Poetry와 pyenv, update-alternatives 연동**

- `pyenv`는 프로젝트별로 독립된 Python 환경을 구성하는 데 유용합니다. 
- `update-alternatives`는 시스템 전역에서 사용할 기본 Python 버전을 선택하는 데 편리합니다.
- Poetry는 `pyproject.toml`에서 지정된 Python 버전을 사용하여 가상 환경을 생성합니다.  만약  `pyproject.toml`에서 지정된 버전이 `pyenv`에 설치되어 있지 않다면, Poetry는 `update-alternatives`에서 설정된 기본 Python 버전을 사용합니다.

**추가 팁:**

- `update-alternatives`는 `python`, `pip` 등 다른 Python 관련 명령어에도 적용할 수 있습니다. 
- 여러 버전을 관리할 때는 버전 충돌에 유의하고, 필요에 따라 `pyenv`와 `update-alternatives`를 적절히 활용하는 것이 좋습니다. 

이제 `pyenv`와 `update-alternatives`를 함께 사용하여 다양한 Python 버전을 관리하고 Poetry를 통해 프로젝트를 효율적으로 개발할 수 있습니다! 😊 궁금한 점은 언제든지 질문하세요! 

<br/>
<br/>

## WSL2에서 Windows Git Credential Manager 사용하기

Windows에 Git이 설치되어 있다면, Windows용으로 제공되는 `git-credential-manager`를 WSL에서도 사용할 수 있습니다. 이렇게 하면 WSL 환경에서 Git을 사용할 때마다 GitHub 계정 정보를 반복해서 입력하지 않아도 됩니다.

**1. Windows 자격 증명 관리자 설정**

- Windows 검색창에 "자격 증명 관리자"를 입력하고 실행합니다.
- "Windows 자격 증명" 탭에서 "일반 자격 증명"을 선택합니다.
- 기존에 GitHub 계정 정보가 등록되어 있지 않다면 "일반 자격 증명 추가"를 클릭하여 다음 정보를 입력합니다.
    - 인터넷 주소 또는 네트워크 주소: `git:https://github.com`
    - 사용자 이름:  GitHub 사용자 이름
    - 암호:  GitHub 비밀번호
- Windows 에 Git Credential Manage 설치
    - [Git Credential Manager](https://git-scm.com)

**2. WSL에서 Git 설정**

- WSL 터미널을 열고  `C:\Program Files\Git\mingw64\bin` 경로에 어떤 파일이 있는지 확인합니다.
- 파일 존재 여부에 따라 다음 명령어 중 하나를 입력하여 `git-credential-manager`를 설정합니다. 

   - **`git-credential-manager.exe` 또는 `git-credential-manager-core.exe` 파일이 존재하는 경우:**

     ```bash
     # git-credential-manager.exe 파일이 존재하는 경우
     git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"

     # git-credential-manager-core.exe 파일이 존재하는 경우
     git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager-core.exe"
     ```

   - **`git-credential-helper-selector.exe` 파일만 존재하는 경우:**

     ```bash
     git config --global credential.helper manager
     ```

**3. user 설정**
  ```bash
  git config --global user.name "herzblue"
  git config --global user.email "herzblue@gmail.com"
  ```

**4. 자격 증명 적용 확인**

- 이제 WSL 환경에서 Git 명령어를 실행할 때, Windows 자격 증명 관리자에 저장된 GitHub 계정 정보가 자동으로 사용됩니다.

**5. 보안**

- PAT (Personal Access Token) 값이 암호화되어 저장되므로, WSL 환경에서 GitHub 계정 정보가 노출될 걱정 없이 안전하게 사용할 수 있습니다. 





<br/>
<br/>

## Google Cloud 설정

## Linux에 gcloud CLI 설치하기

이 가이드는 Linux 시스템에 Google Cloud CLI를 설치하는 방법을 설명합니다. gcloud CLI에는 `gcloud`, `gsutil`, `bq` 명령줄 도구가 포함되어 있습니다.

### 1. Python 버전 확인

Google Cloud CLI는 Python 3.8 ~ 3.12 버전을 필요로 합니다. x86_64 Linux 패키지에는 기본적으로 선호되는 번들 Python 인터프리터가 포함되어 있습니다.  시스템에 지원되는 Python 버전이 설치되어 있는지 확인하세요.

```bash
python3 --version
```

### 2. gcloud CLI 다운로드

Linux 아키텍처에 맞는 gcloud CLI 패키지를 다운로드합니다.

```bash
curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-linux-x86_64.tar.gz
```
- 위 명령어는 64비트 Linux (x86_64) 환경을 위한 것입니다.  
- 다른 아키텍처(Arm, 32비트)를 사용하는 경우  [https://cloud.google.com/sdk/docs/install#linux](https://cloud.google.com/sdk/docs/install#linux) 에서 해당 패키지 URL을 확인하여 명령어를 수정하세요.

### 3. 압축 해제

다운로드한 파일의 압축을 해제합니다.

```bash
tar -xf google-cloud-cli-linux-x86_64.tar.gz
```

### 4. 설치 스크립트 실행

압축을 해제한 디렉토리로 이동하여 설치 스크립트를 실행합니다.

```bash
cd google-cloud-sdk
./install.sh
```

-  설치 과정에서 몇 가지 질문이 표시될 수 있습니다. 
  -  익명 사용 통계 전송 여부를 묻는 질문에는 'Y' 또는 'n'으로 응답합니다.
  -  `PATH` 에 gcloud CLI를 추가하고 명령어 완성을 사용 설정할지 묻는 질문에는 'Y'로 응답하는 것이 좋습니다.

### 5. 터미널 재시작

변경 사항을 적용하려면 터미널을 닫았다가 다시 엽니다.

### 6. gcloud CLI 초기화

gcloud CLI를 초기화하고 Google Cloud 프로젝트를 설정합니다.

```bash
gcloud init
```

-  초기화 과정에서 Google 계정에 로그인하고 프로젝트를 선택하라는 메시지가 표시됩니다.

### 7. 추가 구성 요소 설치 (선택 사항)

필요한 경우,  `gcloud components install`  명령어를 사용하여 추가 구성 요소를 설치할 수 있습니다.

```bash
gcloud components install kubectl
```

### 설치 확인

다음 명령어를 실행하여 gcloud CLI가 제대로 설치되었는지 확인합니다.

```bash
gcloud --version
```

이제 gcloud CLI를 사용하여 Google Cloud Platform 리소스를 관리할 수 있습니다! 😊 












