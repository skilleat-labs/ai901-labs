# Lab 05 — 대화를 기억하는 챗봇 만들기

!!! info "이 실습에서 하는 것"
    지금까지는 **한 번 묻고 끝**이었습니다. 이제 대화가 이어지게 만듭니다.
    핵심은 `messages` 목록 하나입니다.

**선행** [Lab 04](lab04-tokens.md) 완료 · **예상 시간** 35분 · **비용** 약 $0.02 미만

---

## 1. 먼저 문제를 봅니다 — `chat1.py` (8분)

```python
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()
client = OpenAI(
    base_url=os.getenv("AZURE_OPENAI_ENDPOINT"),
    api_key=os.getenv("API_KEY"))
MODEL = os.getenv("MODEL_DEPLOYMENT_NAME")

question = input("질문: ")
r = client.responses.create(model=MODEL, input=question)
print(r.output_text)
```

실행하고 **두 번** 돌려 보세요.

```bash
python chat1.py
# 질문: 내 이름은 홍길동이야.
# → (모델이 인사합니다)

python chat1.py
# 질문: 내 이름이 뭐라고 했지?
# → ???
```

!!! question "모델이 이름을 기억하나요"
    **기억하지 못합니다.** 매번 새로 시작합니다.

    모델은 "기억"이라는 게 없습니다. **매 요청마다 보낸 것만** 봅니다.
    그러니 기억하게 하려면 **지금까지의 대화를 전부 다시 보내야** 합니다.

---

## 2. `chat2.py` — 대화를 쌓습니다 (15분)

```python
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()
client = OpenAI(
    base_url=os.getenv("AZURE_OPENAI_ENDPOINT"),
    api_key=os.getenv("API_KEY"))
MODEL = os.getenv("MODEL_DEPLOYMENT_NAME")

messages = []

while True:
    q = input("질문(종료는 exit): ")
    if q.lower() == "exit":
        break

    messages.append({"role": "user", "content": q})

    r = client.responses.create(
        model=MODEL, input=messages)
    answer = r.output_text
    print(answer)

    messages.append({"role": "assistant", "content": answer})
```

실행합니다.

```bash
python chat2.py
```

이번엔 이렇게 해보세요.

```text
질문: 내 이름은 홍길동이야.
질문: 내 이름이 뭐라고 했지?      ← 이번엔 기억합니다
질문: exit
```

### 무엇이 달라졌나

| | `chat1.py` | `chat2.py` |
|---|---|---|
| 보내는 것 | 문자열 하나 | **목록 전체** |
| `input=` | `"질문"` | `messages` |
| 기억 | 없음 | **목록에 쌓임** |

!!! note "핵심은 두 줄입니다"
    ```python
    messages.append({"role": "user", "content": q})       # 내 질문을 쌓고
    messages.append({"role": "assistant", "content": answer})  # 모델 답도 쌓는다
    ```

    **모델의 답까지 쌓아야** 다음 턴에서 맥락이 이어집니다.
    이 줄을 빼면 모델은 자기가 뭐라고 했는지 모릅니다.

---

## 3. 역할(role) 이해하기 (7분)

`messages` 에 들어가는 각 항목에는 **역할**이 붙습니다.

| role | 누가 | 무엇 |
|---|---|---|
| `system` | 개발자 | **역할·규칙** — 다음 Lab에서 다룹니다 |
| `user` | 사용자 | 그때그때 하는 질문 |
| `assistant` | 모델 | 모델이 한 답 |

!!! note "★ 시험에 이렇게 나옵니다"
    > "챗봇의 말투와 지켜야 할 규칙을 지정하려면 어느 역할에 넣어야 하는가?"

    → **system.** `user` 에 넣으면 그 한 번만 적용되고 다음 턴에 흐려집니다.

---

## 4. 토큰이 늘어나는 걸 확인하세요 (5분)

`chat2.py` 에서 답 출력 뒤에 한 줄을 추가합니다.

```python
    print("누적 토큰:", r.usage.total_tokens)
```

대화를 **5턴 이상** 이어가면서 숫자를 보세요.

!!! warning "대화가 길어질수록 매 턴이 비싸집니다"
    매번 **지금까지의 대화 전체**를 다시 보내기 때문입니다.
    10턴째에는 1턴째보다 훨씬 많은 토큰이 듭니다.

    실무에서는 오래된 대화를 잘라내거나 요약해서 넣습니다.
    **컨텍스트 창(context window)** 한도를 넘으면 오류가 납니다.

---

## 5. ✅ 체크포인트

!!! success "여기까지 됐으면 통과입니다"
    - [ ] `chat1.py` 가 이름을 **기억하지 못하는 것**을 확인했다
    - [ ] `chat2.py` 가 이름을 **기억하는 것**을 확인했다
    - [ ] `messages` 에 `user` 와 `assistant` 를 **둘 다** 쌓는 이유를 말할 수 있다
    - [ ] 대화가 길어질수록 **누적 토큰이 늘어나는 것**을 확인했다

---

## 🔧 안 될 때

??? failure "두 번째 질문에서도 기억을 못 합니다"
    `messages.append({"role": "assistant", ...})` 줄이 빠졌거나,
    `while` 문 **바깥**에 있는지 확인하세요. 들여쓰기가 맞아야 합니다.

??? failure "input=messages 에서 오류가 납니다"
    `messages` 가 **목록**인지 확인하세요. `messages = []` 로 시작해야 합니다.
    그리고 각 항목은 `{"role": ..., "content": ...}` 형태여야 합니다.

??? failure "종료가 안 됩니다"
    `exit` 를 입력하세요. 그래도 안 되면 ++ctrl+c++ 를 누르면 강제 종료됩니다.

??? failure "대화가 길어지니 오류가 납니다"
    컨텍스트 창 한도를 넘었을 수 있습니다. 프로그램을 다시 시작하세요.
    실무에서는 오래된 메시지를 잘라냅니다.

---

## 💡 오늘 기억할 한 줄

!!! quote ""
    **모델에는 기억이 없습니다. `messages` 목록이 기억입니다.**
