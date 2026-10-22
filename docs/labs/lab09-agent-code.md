# Lab 09 — 코드에서 에이전트 호출하기

!!! info "이 실습에서 하는 것"
    포털에서 만든 `hr-agent` 를 **내 코드에서** 부릅니다.
    여기서 인증 방식이 바뀝니다 — **키가 아니라 ID 인증**입니다.

**선행** [Lab 08](lab08-agent-portal.md) 완료 · **예상 시간** 36분 · **비용** 약 $0.05 미만

---

## 1. Azure CLI 로그인 (10분)

### 왜 키를 안 쓰나

| | 모델 호출 (Lab 04) | 에이전트 호출 (지금) |
|---|---|---|
| 인증 | **API 키** | **ID (자격 증명)** |
| 코드 | `api_key=...` | `DefaultAzureCredential()` |
| 이유 | 모델만 부르면 됨 | 프로젝트 전체 권한이 필요 |

!!! note "★ 시험에 그대로 나옵니다"
    "에이전트를 코드로 호출할 때 사용하는 인증 방식은?" → **DefaultAzureCredential (ID 인증)**
    `api_key=` 를 넣는 선택지가 오답으로 섞여 나옵니다.

### 설치 확인

```bash
az --version
```

??? failure "명령을 찾을 수 없다면 — 지금 설치"
    === "Windows"
        ```powershell
        winget install -e --id Microsoft.AzureCLI
        ```
        설치 후 **터미널을 닫고 새로 여세요.**

    === "macOS"
        ```bash
        brew install azure-cli
        ```

    설치가 오래 걸리면 손을 드세요. 옆 사람 화면으로 먼저 따라오셔도 됩니다.

### 로그인

```bash
az login
```

브라우저가 열립니다. **Lab 00에서 쓴 학교 계정**으로 로그인하세요.

```bash
az account show
```

구독 정보가 나오면 성공입니다.

!!! warning "다른 계정으로 로그인하면 에이전트를 못 찾습니다"
    `az account show` 의 `user` 가 포털에서 쓰는 계정과 같은지 확인하세요.

---

## 2. 패키지 설치 (3분)

```bash
pip install azure-identity azure-ai-projects
```

!!! tip "`(.venv)` 가 보이는지 먼저 확인"
    가상 환경이 꺼져 있으면 엉뚱한 곳에 설치됩니다.

---

## 3. 프로젝트 엔드포인트 확인 (5분)

!!! danger "★ 지금까지 쓰던 엔드포인트와 다릅니다"
    | | 어디서 쓰나 | 생김새 |
    |---|---|---|
    | **리소스 엔드포인트** | 모델 호출 (Lab 04) | `...openai.azure.com/...` |
    | **프로젝트 엔드포인트** | **에이전트 호출 (지금)** | 프로젝트 개요에 별도 표시 |

    둘을 바꿔 넣으면 연결이 안 됩니다. **시험에도 나오는 구분입니다.**

1. [ai.azure.com](https://ai.azure.com) → 내 프로젝트 → **개요**
2. **프로젝트 엔드포인트**를 복사합니다
3. `.env` 에 **한 줄 추가**합니다

```ini
PROJECT_ENDPOINT=여기에_프로젝트_엔드포인트_붙여넣기
```

---

## 4. `call_agent.py` (13분)

```python
import os
from dotenv import load_dotenv
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient

load_dotenv()

project_client = AIProjectClient(
    endpoint=os.getenv("PROJECT_ENDPOINT"),
    credential=DefaultAzureCredential())

agent = project_client.agents.get("hr-agent")
openai_client = project_client.get_openai_client()

response = openai_client.responses.create(
    input=[{"role": "user", "content": "휴가 신청 어떻게 하나요?"}],
    extra_body={"agent": {"name": agent.name,
                          "type": "agent_reference"}})

print(response.output_text)
```

```bash
python call_agent.py
```

### 줄별로 하는 일

| 줄 | 하는 일 |
|---|---|
| `DefaultAzureCredential()` | `az login` 한 신원을 **자동으로 찾아** 씁니다 |
| `AIProjectClient(...)` | **프로젝트**에 연결합니다 (모델이 아니라) |
| `agents.get("hr-agent")` | 포털에서 만든 에이전트를 **가져옵니다** |
| `get_openai_client()` | 프로젝트 인증을 **물려받은** OpenAI 클라이언트 |
| `extra_body={"agent": ...}` | "이 요청은 **이 에이전트가** 처리해라" |

!!! question "왜 `OpenAI(...)` 를 새로 안 만드나요"
    `get_openai_client()` 는 프로젝트의 인증을 그대로 물려받습니다.
    새로 만들면 키가 또 필요해집니다. **★ 시험 함정 지점입니다.**

---

## 5. 모델 호출과 나란히 보기 (5분)

`call_model.py`(Lab 04)와 `call_agent.py`를 양쪽에 띄워 놓고 비교하세요.

| | 모델 호출 | 에이전트 호출 |
|---|---|---|
| 클라이언트 | `OpenAI(base_url, api_key)` | `AIProjectClient(endpoint, credential)` |
| 엔드포인트 | 리소스 | **프로젝트** |
| 인증 | API 키 | **DefaultAzureCredential** |
| 지침 | 코드의 `SYSTEM` | **포털에 저장됨** |
| 지식 | 프롬프트에 붙임 | **업로드한 파일** |

!!! note "★ 시험 요령 — 다른 줄만 외우세요"
    두 코드는 대부분 같습니다. **다른 네 줄**이 곧 출제 지점입니다.
    `DefaultAzureCredential` · `AIProjectClient` · `get_openai_client` · `agent_reference`

---

## 6. ✅ 체크포인트

!!! success "여기까지 됐으면 통과입니다"
    - [ ] `az account show` 가 **포털과 같은 계정**을 보여준다
    - [ ] `.env` 에 `PROJECT_ENDPOINT` 가 추가되어 있다
    - [ ] `call_agent.py` 가 **근거 조항과 함께** 답한다
    - [ ] 모델 호출과 에이전트 호출의 **차이 네 가지**를 말할 수 있다

---

## 🔧 안 될 때

??? failure "DefaultAzureCredential 인증 실패"
    가장 흔합니다. 순서대로 확인하세요.

    1. `az login` 을 **했는지**
    2. `az account show` 의 계정이 **포털 계정과 같은지**
    3. 여러 계정에 로그인돼 있다면: `az account set --subscription "<구독ID>"`
    4. 터미널을 **새로 열고** 다시 실행

??? failure "에이전트를 찾을 수 없습니다 (hr-agent)"
    - 포털에서 에이전트 이름이 **정확히 `hr-agent`** 인지 확인 (대소문자·공백)
    - `PROJECT_ENDPOINT` 가 **프로젝트** 엔드포인트인지 확인 (리소스 것 아님)
    - 에이전트가 **저장**되었는지 확인

??? failure "ModuleNotFoundError: azure.ai.projects"
    ```bash
    pip install azure-identity azure-ai-projects
    ```
    `(.venv)` 가 켜진 상태에서 실행했는지 확인하세요.

??? failure "권한 오류 (Forbidden / 403)"
    로그인한 계정이 프로젝트에 접근 권한이 없습니다.

    - 포털에서 그 프로젝트가 **보이는 계정**으로 `az login` 했는지 확인
    - 강사 구독을 쓰는 경우라면 권한 부여가 필요합니다. 손을 드세요

??? failure "extra_body 에서 오류가 납니다"
    SDK 버전 차이일 수 있습니다. 오류 메시지를 보고 손을 드세요.
    급하면 포털 플레이그라운드로 대체하고 넘어갑니다 — 개념은 동일합니다.

??? failure "az login 에서 브라우저가 안 열립니다"
    ```bash
    az login --use-device-code
    ```
    화면에 나온 코드를 브라우저에 입력하는 방식입니다.
