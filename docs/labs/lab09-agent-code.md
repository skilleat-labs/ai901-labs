# Lab 09 — 코드에서 에이전트 호출하기

!!! info "이 실습에서 하는 것"
    포털에서 만든 `hr-agent` 를 **내 코드에서** 부릅니다.
    여기서 인증 방식이 바뀝니다 — **키가 아니라 ID 인증**입니다.

**선행** [Lab 08](lab08-agent-portal.md) 완료 · **예상 시간** 36분 (Azure CLI 설치가 필요하면 +10분) · **비용** 약 $0.05 미만

---

## 1. Azure CLI 설치·로그인 — 다 같이 (10분, 설치가 필요하면 +10분)

### 왜 키를 안 쓰나

| | 모델 호출 (Lab 04) | 에이전트 호출 (지금) |
|---|---|---|
| 이 과정에서 쓴 방식 | **API 키** | **ID (자격 증명)** |
| 코드 | `api_key=...` | `DefaultAzureCredential()` |
| 무엇에 연결하나 | 리소스의 모델 | **프로젝트** |

!!! note "★ 시험에 그대로 나옵니다"
    "에이전트를 코드로 호출할 때 사용하는 인증 방식은?" → **DefaultAzureCredential (ID 인증)**
    `api_key=` 를 넣는 선택지가 오답으로 섞여 나옵니다.

!!! warning "다만 — “모델은 키, 에이전트는 ID” 는 아닙니다"
    오해하기 쉬운 지점이라 정확히 짚고 갑니다.

    **모델 호출도 Microsoft Entra ID 로 할 수 있습니다.** 공식 문서는 오히려
    **프로덕션에서는 키보다 Entra ID 를 권장**합니다. 모델 호출에 ID 를 쓰면 이런 모양입니다.

    ```python
    from azure.identity import DefaultAzureCredential, get_bearer_token_provider

    token_provider = get_bearer_token_provider(
        DefaultAzureCredential(), "https://ai.azure.com/.default")
    client = OpenAI(base_url=endpoint, api_key=token_provider)
    ```

    | | 키 인증 | Entra ID 인증 |
    |---|---|---|
    | 모델 호출 | 가능 | **가능 (권장)** |
    | 에이전트 호출 | — | **이 방식만** |

    **정확한 구분은 이렇습니다.**
    모델 호출은 **둘 다** 되고, 에이전트(프로젝트)는 **ID 인증이 필요**합니다.

    우리가 Lab 02~07에서 키를 쓴 건 **처음 배울 때 단계가 적어서**입니다.

### 설치 확인 — 여기서 함께 설치합니다

```bash
az --version
```

버전이 여러 줄 나오면 설치는 끝난 상태입니다. 아래 **로그인** 으로 바로 가세요.

!!! info "“명령을 찾을 수 없음” 이 나오면 — 지금 다 같이 설치합니다 (5~10분)"
    사전에 설치해 오신 분이 없어도 괜찮습니다. **이 랩 시간에 포함된 단계입니다.**
    설치가 끝난 분은 먼저 로그인까지 해두고 기다려 주세요.

    === "Windows"
        ```powershell
        winget install -e --id Microsoft.AzureCLI
        ```
        설치 후 **터미널을 닫고 새로 여세요.**

    === "macOS"
        ```bash
        brew install azure-cli
        ```

    설치가 끝나면 `az --version` 으로 다시 확인합니다.
    오래 걸리거나 막히면 손을 드세요. 옆 사람 화면으로 먼저 따라오셔도 됩니다.

### 로그인

```bash
az login
```

브라우저가 열립니다. **배부받은 계정**(포털에서 쓰는 그 계정)으로 로그인하세요.

```bash
az account show
```

구독 정보가 나오면 성공입니다.

!!! warning "다른 계정으로 로그인하면 에이전트를 못 찾습니다"
    `az account show` 의 `user` 가 포털에서 쓰는 계정과 같은지 확인하세요.

---

## 2. 패키지 설치 (3분)

Lab 02에서 만든 `requirements.txt` 에 **두 줄을 추가**합니다.

```ini title="requirements.txt"
openai==2.46.0
python-dotenv==1.2.4
azure-identity==1.25.3
azure-ai-projects==2.9.0
```

```bash
pip install -r requirements.txt
```

!!! tip "`(.venv)` 가 보이는지 먼저 확인"
    가상 환경이 꺼져 있으면 엉뚱한 곳에 설치됩니다.

!!! warning "버전을 고정한 이유"
    `azure-ai-projects` 는 아직 변화가 빠른 패키지입니다.
    버전이 올라가면 `agents.get()` 이나 `extra_body` 의 형태가 달라질 수 있어서
    **교실 전체를 같은 버전으로** 맞춥니다.

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
    **가장 흔한 원인은 이름입니다.** 새 빌더는 에이전트를 만들 때 이름을 묻지 않고
    `silver-agent-hxmlzpz35c` 처럼 **임의의 이름을 자동으로 붙입니다.**
    Lab 08 에서 이름을 바꾸지 않았다면 여기서 반드시 막힙니다.

    1. 포털 → 에이전트 → 이름이 **정확히 `hr-agent`** 인지 확인 (대소문자·공백)
       - 다르면 이름을 클릭하거나 **세부 정보** 탭에서 바꾸고 저장
    2. `PROJECT_ENDPOINT` 가 **프로젝트** 엔드포인트인지 확인 (리소스 것 아님)
    3. 에이전트가 **저장**되었는지 확인

??? failure "ModuleNotFoundError: azure.ai.projects"
    ```bash
    pip install -r requirements.txt
    ```
    `(.venv)` 가 켜진 상태에서 실행했는지 확인하세요.
    `requirements.txt` 에 `azure-ai-projects==2.9.0` 줄을 추가했는지도 보세요.

??? failure "권한 오류 (Forbidden / 403)"
    **키 인증과 ID 인증의 차이가 여기서 드러납니다.**

    Lab 02~07 은 키로 호출해서 역할을 거치지 않았습니다.
    지금은 ID 인증이라 **데이터 평면 역할**이 있어야 합니다.

    | 역할 | 담당 | Lab 09 에 필요? |
    |---|---|---|
    | `Contributor` | 제어 평면 (리소스 생성) | 부족함 |
    | **`Cognitive Services User`** | **데이터 평면 (추론·에이전트)** | **필요** |

    1. 포털에서 그 프로젝트가 **보이는 계정**으로 `az login` 했는지 확인
    2. `az account show` 의 계정이 포털 계정과 같은지 확인
    3. 그래도 403 이면 **역할이 누락된 것입니다. 손을 드세요** —
       강사가 `Cognitive Services User` 를 부여하면 1~2분 뒤 됩니다

??? failure "extra_body 에서 오류가 납니다"
    SDK 버전 차이일 수 있습니다. 오류 메시지를 보고 손을 드세요.
    급하면 포털 플레이그라운드로 대체하고 넘어갑니다 — 개념은 동일합니다.

??? failure "az login 에서 브라우저가 안 열립니다"
    ```bash
    az login --use-device-code
    ```
    화면에 나온 코드를 브라우저에 입력하는 방식입니다.
