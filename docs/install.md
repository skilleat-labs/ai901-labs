# 수업 전 설치 — 노트북 준비

!!! info "언제 하나요"
    **개강 전에 집에서** 해두시면 됩니다. **10~15분**이면 끝납니다.
    교실에서 32명이 동시에 설치하면 오래 걸려서, 미리 해두시면 1일차가 훨씬 수월합니다.

!!! success "못 하셔도 괜찮습니다 — 그냥 오세요"
    1일차 오전에 다 같이 확인하고, 안 되면 그 자리에서 도와드립니다.
    끝까지 안 되면 **브라우저만으로** 실습하는 방법도 준비돼 있습니다.

    미리 하시면 **편해진다**는 것뿐입니다. 혼자 오래 붙잡고 있지 마세요.

## 설치할 것

| | 무엇 | 언제 쓰나 | 시간 |
|---|---|---|---|
| 1 | **Python 3.10 이상** | 1일차 오후부터 4일 내내 | 10분 |
| 2 | **VS Code** | 코드를 쓰고 저장하는 편집기 | 3분 |
| 3 | Azure CLI | **3일차 수업 중에 같이 합니다** — 미리 해두면 편함 | 5분 |

!!! note "Azure 계정은 준비하지 않으셔도 됩니다"
    교육용 계정을 만들어서 **1일차에 나눠 드립니다.**
    가입 절차도, **신용카드 등록도 없습니다.**

    혹시 어느 화면에서든 카드 번호를 요구하면 **멈추고 강사에게 알려주세요.**

---

## 1. Python 설치 (10분)

=== "Windows"

    **① 내려받기**

    [python.org/downloads](https://www.python.org/downloads/) 에 접속하면
    가운데 노란 버튼에 **「Download Python 3.x.x」** 가 있습니다. 누릅니다.

    **② 설치 — 여기가 제일 중요합니다**

    내려받은 파일을 실행하면 첫 화면이 뜹니다.

    ```text
    ┌────────────────────────────────────────────┐
    │  Install Python 3.13                       │
    │                                            │
    │     [ Install Now ]                        │
    │     [ Customize installation ]             │
    │                                            │
    │  ☑ Use admin privileges when installing    │
    │  ☑ Add python.exe to PATH   ← 반드시 체크!  │
    └────────────────────────────────────────────┘
    ```

    !!! danger "「Add python.exe to PATH」를 반드시 체크하세요"
        화면 **맨 아래**에 작게 있습니다. 기본값이 **꺼져 있습니다.**

        이걸 놓치면 설치는 되는데 **터미널에서 `python` 명령이 안 먹습니다.**
        매 기수 가장 많이 나는 문제입니다.

        체크하고 **Install Now** 를 누르세요.

    **③ 확인 — 터미널을 새로 엽니다**

    설치가 끝나면 **PowerShell 을 새로 엽니다.** (이미 열려 있었다면 닫고 다시)

    - 시작 버튼 → `powershell` 입력 → **Windows PowerShell** 실행

    ```powershell
    python --version
    ```

    ```text
    Python 3.13.1     ← 이렇게 나오면 성공 (3.10 이상이면 됩니다)
    ```

=== "macOS"

    **① 내려받기**

    [python.org/downloads](https://www.python.org/downloads/) 에서
    **「Download Python 3.x.x」** 를 누릅니다. `.pkg` 파일이 받아집니다.

    **② 설치**

    받은 파일을 열고 **계속 → 계속 → 설치** 로 끝까지 진행합니다.
    맥에서는 PATH 설정이 자동이라 따로 체크할 게 없습니다.

    **③ 확인**

    **터미널**을 엽니다. (`Command + Space` → `터미널` 입력)

    ```bash
    python3 --version
    ```

    ```text
    Python 3.13.1     ← 이렇게 나오면 성공 (3.10 이상이면 됩니다)
    ```

    !!! tip "맥은 `python` 이 아니라 `python3` 입니다"
        맥에 기본으로 깔린 옛 버전과 구분하려고 그렇습니다.
        실습 중에도 `python` 이 안 되면 `python3` 로 바꿔서 해보세요.

    ??? question "Homebrew 를 쓰신다면"
        ```bash
        brew install python
        ```
        이것도 됩니다. 편한 쪽으로 하세요.

---

## 2. VS Code 설치 (3분)

코드를 쓰고 저장하는 편집기입니다. 메모장으로도 되지만 훨씬 편합니다.

[code.visualstudio.com](https://code.visualstudio.com) → **Download** → 설치

기본 설정 그대로 **다음 → 다음** 으로 끝내시면 됩니다.

!!! tip "한국어로 쓰고 싶다면"
    설치 후 VS Code 를 열면 오른쪽 아래에 한국어 팩을 설치하겠냐고 물어봅니다.
    **설치하고 다시 시작**하면 메뉴가 한국어가 됩니다.

---

## 3. Azure CLI — 여유가 되시면 (5분)

**3일차 Lab 09** 에서 씁니다. **숙제가 아닙니다** — Lab 09 맨 앞에서 다 같이 설치합니다.
미리 해두시면 그만큼 빨리 넘어간다는 것뿐입니다.

=== "Windows"

    PowerShell 에서:

    ```powershell
    winget install -e --id Microsoft.AzureCLI
    ```

    설치 후 **터미널을 닫고 새로 연 뒤**:

    ```powershell
    az --version
    ```

=== "macOS"

    ```bash
    brew install azure-cli
    ```

    ```bash
    az --version
    ```

버전 정보가 여러 줄 출력되면 성공입니다.

---

## ✅ 최종 확인

터미널(Windows 는 PowerShell)에서 아래 두 줄이 되면 준비 끝입니다.

=== "Windows"

    ```powershell
    python --version
    ```
    ```text
    Python 3.10 이상
    ```

=== "macOS"

    ```bash
    python3 --version
    ```
    ```text
    Python 3.10 이상
    ```

---

## 🔧 안 될 때

??? failure "“python을(를) 찾을 수 없습니다” / “command not found”"
    **대부분 PATH 문제**입니다. 설치는 됐는데 터미널이 못 찾는 상태예요.

    === "Windows"
        1. **터미널을 닫고 새로 열어** 보세요 — 이것만으로 되는 경우가 많습니다
        2. `py --version` 으로도 시도해 보세요
        3. 그래도 안 되면 **다시 설치**합니다.
           설치 화면에서 **「Add python.exe to PATH」 체크**를 꼭 하세요
           (이미 깔려 있으면 `Modify` 가 뜹니다 → `Repair` 를 고르면 됩니다)

    === "macOS"
        `python` 말고 **`python3`** 로 해보세요. 맥은 `python3` 가 정상입니다.

??? failure "버전이 3.10 보다 낮게 나옵니다 (3.8, 3.9 등)"
    예전에 설치한 버전이 남아 있는 것입니다.
    [python.org](https://www.python.org/downloads/) 에서 **최신 버전을 새로 설치**하세요.
    옛 버전을 지우지 않아도 됩니다.

??? failure "Windows — 설치할 때 권한 오류가 납니다"
    회사·학교 노트북이면 설치 권한이 막혀 있을 수 있습니다.

    1. 설치 파일을 **마우스 오른쪽 → 관리자 권한으로 실행**
    2. 그래도 안 되면 **설치하지 마시고 그냥 오세요.**
       브라우저만으로 실습하는 방법을 안내해 드립니다

??? failure "맥 — “확인되지 않은 개발자” 경고가 뜹니다"
    python.org 에서 받은 파일이면 안전합니다.

    **시스템 설정 → 개인정보 보호 및 보안** 으로 가서 아래쪽의
    **「확인 없이 열기」** 를 누르면 진행됩니다.

??? failure "뭘 해도 안 됩니다"
    **그냥 오세요.** 진심입니다.

    1일차 오전 Lab 00 에서 다 같이 확인하고 도와드립니다.
    끝까지 안 되면 **브라우저에서 하는 방법**이 있어서 수업은 따라가실 수 있습니다.

    혼자 30분 이상 붙잡고 계시지 마세요.

---

## 다음

설치가 끝나셨으면 [시작하기](00-setup.md) 를 한 번 훑어보시면 됩니다.
나머지는 **1일차에 다 같이** 합니다.
