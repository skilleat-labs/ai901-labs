# Lab 04 — 첫 호출 확장과 토큰 사용량

!!! info "이 실습에서 하는 것"
    어제 만든 `verify.py` 를 확장해서, **토큰이 얼마나 쓰였는지** 눈으로 확인합니다.
    토큰은 곧 **비용**입니다.

**선행** [Lab 03](lab03-deploy-model.md) 완료 · **예상 시간** 30분 · **비용** 약 $0.01 미만

---

## 1. 어제 폴더로 돌아가기 (3분)

```bash
cd ai901
```

=== "Windows"
    ```powershell
    .venv\Scripts\activate
    ```
=== "macOS · Linux"
    ```bash
    source .venv/bin/activate
    ```

프롬프트 앞에 **`(.venv)`** 가 보이는지 확인하세요.

!!! warning "매일 아침 이 두 줄로 시작합니다"
    터미널을 새로 열면 가상 환경이 꺼집니다. 4일 내내 가장 많이 나는 오류입니다.

---

## 2. `call_model.py` 만들기 (12분)

같은 폴더에 새 파일을 만듭니다.

```python
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()
client = OpenAI(
    base_url=os.getenv("AZURE_OPENAI_ENDPOINT"),
    api_key=os.getenv("API_KEY"))

response = client.responses.create(
    model=os.getenv("MODEL_DEPLOYMENT_NAME"),
    input="튜링 테스트가 무엇인지 3문장으로 설명해줘.")

print(response.output_text)
print("토큰:", response.usage)
```

실행합니다.

```bash
python call_model.py
```

### 어제와 달라진 점

| | 어제 `verify.py` | 오늘 `call_model.py` |
|---|---|---|
| 목적 | **연결이 되는가** | **실제로 쓸 수 있는가** |
| 프롬프트 | "인사해 주세요" | 의미 있는 질문 |
| 출력 | 답만 | 답 **+ 토큰 사용량** |

---

## 3. 토큰 읽기 (10분)

출력된 `usage` 를 봅니다. 이런 형태입니다.

```text
토큰: ResponseUsage(input_tokens=24, output_tokens=118, total_tokens=142)
```

| 항목 | 의미 |
|---|---|
| `input_tokens` | **내가 보낸** 프롬프트의 토큰 수 |
| `output_tokens` | **모델이 만든** 답의 토큰 수 |
| `total_tokens` | 둘의 합 — **TPM이 세는 게 이것입니다** |

### 직접 확인해 보세요

프롬프트를 바꿔 가며 토큰이 어떻게 변하는지 기록합니다.

| 프롬프트 | input | output | total |
|---|---|---|---|
| `"안녕"` | | | |
| `"튜링 테스트가 무엇인지 3문장으로 설명해줘."` | | | |
| `"튜링 테스트가 무엇인지 자세히 설명해줘."` (3문장 제한 없이) | | | |

!!! question "무엇이 보이나요"
    - 입력을 조금 늘려도 **출력 토큰이 훨씬 많이** 늘어납니다
    - **"3문장으로"** 같은 제한 한 마디가 출력 토큰을 크게 줄입니다
    - 비용을 줄이는 가장 쉬운 방법은 **답 길이를 제한하는 것**입니다

!!! tip "한국어는 영어보다 토큰을 더 먹습니다"
    같은 뜻이라도 한국어가 토큰 수가 많습니다. 비용 계산할 때 염두에 두세요.

---

## 4. ✅ 체크포인트

!!! success "여기까지 됐으면 통과입니다"
    - [ ] `call_model.py` 가 모델의 답을 출력했다
    - [ ] `usage` 에 input / output / total 토큰이 보인다
    - [ ] 프롬프트를 **3개 이상** 바꿔가며 토큰 변화를 기록했다
    - [ ] "3문장으로" 같은 제한이 출력 토큰을 줄인다는 걸 확인했다

---

## 🔧 안 될 때

??? failure "ModuleNotFoundError 가 납니다"
    가상 환경이 꺼져 있습니다. 1절의 활성화 명령을 다시 실행하세요.
    프롬프트 앞에 `(.venv)` 가 있어야 합니다.

??? failure "404 Not Found"
    `.env` 의 `MODEL_DEPLOYMENT_NAME` 이 포털의 **배포 이름**과 같은지 확인하세요.
    Lab 03에서 배포를 바꿨다면 이름도 바뀌었을 수 있습니다.

??? failure "usage 가 None 이거나 비어 있습니다"
    SDK 버전에 따라 속성 이름이 다를 수 있습니다. 아래로 바꿔 보세요.

    ```python
    print("입력:", response.usage.input_tokens)
    print("출력:", response.usage.output_tokens)
    ```

    그래도 안 되면 `print(response)` 로 전체 응답을 찍어 구조를 확인하세요.

??? failure "429 Too Many Requests"
    교실에서 동시에 몰렸습니다. 30초 뒤 재시도하거나,
    [Lab 03](lab03-deploy-model.md) 3절에서 TPM을 올리세요.

---

## 💡 시험에 나오는 지점

!!! note "★ 이 코드의 구조가 그대로 출제됩니다"
    | 질문 | 답 |
    |---|---|
    | "프롬프트를 **제출하는 줄**은?" | `client.responses.create(...)` |
    | "`OpenAI(...)` 가 하는 일은?" | **연결을 만든다** — 아직 아무것도 안 보냅니다 |
    | "`model=` 에 넣는 값은?" | 모델 이름이 아니라 **배포 이름** |
    | "TPM이 세는 것은?" | **입력 + 출력 토큰의 합** |
