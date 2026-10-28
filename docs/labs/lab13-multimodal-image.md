# Lab 13 — 멀티모달 이미지 분석

!!! info "이 실습에서 하는 것"
    모델에게 **글자와 이미지를 같이** 보냅니다.
    이미지를 넘기는 **두 가지 방법**을 둘 다 해봅니다 — 시험에 나옵니다.

**선행** [Lab 04](lab04-tokens.md) 완료 · **예상 시간** 36분 · **비용** 약 $0.05 미만

---

## 0. 무엇이 달라지나 (4분)

지금까지 `input` 에는 **글자만** 넣었습니다.

```python
input="튜링 테스트가 뭐야?"                      # 문자열
input=[{"role": "user", "content": "..."}]      # 역할 있는 목록
```

멀티모달은 `content` 가 **목록**이 됩니다.

```python
content=[
    {"type": "input_text",  "text": "이 사진을 설명해줘."},
    {"type": "input_image", "image_url": "..."}
]
```

!!! note "★ 핵심 — `content` 안에 여러 부분을 넣습니다"
    `input_text` 와 `input_image` 를 **나란히** 넣습니다.
    이미지를 여러 장 보내려면 `input_image` 를 여러 개 넣으면 됩니다.

---

## 1. 준비 — 이미지 내려받기 (3분)

실습용 이미지를 준비해 뒀습니다. **받아서 `ai901` 폴더에 넣으면 됩니다.**

<figure markdown>
  ![AI-901 합격 기원](../assets/sample.png){ width="420" }
  <figcaption>sample.png · 800 × 560 · 약 105KB</figcaption>
</figure>

[↓ sample.png 내려받기](../assets/sample.png){ .md-button .md-button--primary download="sample.png" }

### 받은 다음 — `ai901` 폴더로 옮깁니다

=== "Windows"

    ```powershell
    # 보통 다운로드 폴더에 받아집니다
    move "$env:USERPROFILE\Downloads\sample.png" .
    dir sample.png
    ```

=== "macOS · Linux"

    ```bash
    mv ~/Downloads/sample.png .
    ls -l sample.png
    ```

!!! warning "터미널 위치가 `ai901` 인지 먼저 확인하세요"
    위 명령의 `.` 은 **지금 터미널이 있는 폴더**라는 뜻입니다.
    `pwd` (Windows 는 `cd`) 를 쳐서 경로 끝이 `ai901` 인지 보세요.
    `analyze_image.py` 와 **같은 폴더**에 있어야 합니다.

!!! tip "이 이미지를 고른 이유 — 글자가 들어 있습니다"
    아래 프롬프트가 "글자가 있으면 읽어줘" 입니다.
    모델이 캐릭터를 **설명**하면서 「합격」 「AI-901 합격 기원」 까지 **읽어내는지** 보세요.
    그림만 있는 사진으로는 이걸 확인할 수 없습니다.

??? tip "내 사진을 쓰고 싶다면"
    휴대폰 사진, 스크린샷 아무거나 괜찮습니다. 파일명만 **`sample.png`** 로 맞추세요.

    - **1MB 이하**를 권합니다 — 크면 느리고 토큰이 많이 듭니다
    - **글자가 있는 것**을 고르세요. 영수증·간판·표지판이 좋습니다
    - JPG 를 쓰면 뒤에 나오는 `data:image/png;base64,` 를 `data:image/jpeg;base64,` 로 바꿔야 합니다

    영수증을 쓰면 Lab 15(정보 추출)와 자연스럽게 이어집니다.

---

## 2. 방법 ① base64 — 내 컴퓨터 파일 (14분)

`analyze_image.py` 를 만듭니다.

```python
import os, base64
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()
client = OpenAI(
    base_url=os.getenv("AZURE_OPENAI_ENDPOINT"),
    api_key=os.getenv("API_KEY"))

with open("./sample.png", "rb") as image_file:
    image = base64.b64encode(image_file.read()).decode("utf-8")

response = client.responses.create(
    model=os.getenv("MODEL_DEPLOYMENT_NAME"),
    input=[{"role": "user", "content": [
        {"type": "input_text",
         "text": "이 이미지를 한국어로 설명해줘. 글자가 있으면 읽어줘."},
        {"type": "input_image",
         "image_url": f"data:image/png;base64,{image}"}
    ]}])

print(response.output_text)
```

```bash
python analyze_image.py
```

### 세 줄을 뜯어봅니다

| 줄 | 하는 일 |
|---|---|
| `base64.b64encode(...)` | 파일을 **문자열로** 바꿉니다 |
| `.decode("utf-8")` | bytes → str (이게 빠지면 오류) |
| `f"data:image/png;base64,{image}"` | **data URI** 형식으로 감쌉니다 |

!!! warning "`data:image/png;base64,` 를 빠뜨리면 안 됩니다"
    인코딩된 문자열만 넣으면 모델이 못 읽습니다. **접두사가 형식을 알려주는 역할**입니다.
    JPG면 `data:image/jpeg;base64,` 로 바꾸세요.

---

## 3. 방법 ② URL — 공개된 이미지 (10분)

같은 파일에서 이미지 부분만 바꿔 봅니다.

```python
response = client.responses.create(
    model=os.getenv("MODEL_DEPLOYMENT_NAME"),
    input=[{"role": "user", "content": [
        {"type": "input_text", "text": "이 이미지를 한국어로 설명해줘."},
        {"type": "input_image",
         "image_url": "https://upload.wikimedia.org/wikipedia/commons/3/3a/Cat03.jpg"}
    ]}])
```

!!! tip "같은 이미지로 비교해 보고 싶다면"
    방금 내려받은 `sample.png` 는 **실습 사이트에 그대로 올라가 있습니다.** 이 주소를 쓰면
    **같은 이미지를 두 방법으로** 보낼 수 있습니다.

    ```python
    "image_url": "https://skilleat-labs.github.io/ai901-labs/assets/sample.png"
    ```

    base64 로 보낸 답과 **비슷하게 나오는지** 보세요. 전달 방법이 달라도
    모델이 보는 이미지는 같습니다 — 그래서 답도 같은 내용이 나옵니다.

!!! danger "★★ 여기가 시험 함정입니다"
    **`image_url` 이라는 필드는 하나뿐입니다.** 거기에

    - 공개 URL 을 넣거나
    - base64 data URI 를 넣거나

    **둘 중 하나**를 넣습니다. **URL 전용 필드가 따로 있는 게 아닙니다.**

    2026년 8월 응시자가 "지미하게 걸리는 포인트"로 지목한 지점입니다.

### 언제 무엇을 쓰나

| | URL | base64 |
|---|---|---|
| 조건 | 인터넷에서 **공개 접근 가능**해야 함 | 로컬 파일이면 됨 |
| 요청 크기 | 주소 한 줄 — 가벼움 | **원본 파일보다 약 33% 커짐** |
| 쓰는 경우 | 이미 웹에 올라간 이미지 | 사내 문서 · 개인 사진 |

!!! question "왜 base64가 33% 커지나요"
    바이너리 3바이트를 글자 4개로 바꾸기 때문입니다 (4 ÷ 3 ≈ 1.33).

!!! warning "★ “33% 커지니까 토큰도 33% 늘어난다”가 아닙니다"
    혼동하기 쉬운 지점입니다. **커지는 것은 전송량(요청 본문 크기)** 입니다.

    | | 무엇이 결정하나 |
    |---|---|
    | **전송량** | base64 문자열 길이 — 원본의 약 1.33배 |
    | **이미지 토큰** | **이미지의 가로·세로 크기**(해상도) |

    모델은 이미지를 **타일로 나눠서** 토큰을 계산합니다.
    같은 사진을 URL 로 보내든 base64 로 보내든 **이미지 토큰은 같습니다.**

    그래서 토큰을 줄이려면 **인코딩 방식을 바꾸는 게 아니라 해상도를 줄여야** 합니다.

---

## 4. ✅ 체크포인트

!!! success "여기까지 됐으면 통과입니다"
    - [ ] **base64 방식**으로 이미지 설명을 받았다
    - [ ] **URL 방식**으로도 받았다
    - [ ] `image_url` 필드 **하나**에 둘 다 들어간다는 걸 확인했다
    - [ ] `content` 가 `input_text` + `input_image` **목록**이라는 걸 안다

---

## 🔧 안 될 때

??? failure "이미지를 인식하지 못한다고 합니다"
    배포한 모델이 **멀티모달**인지 확인하세요.
    `gpt-5.4-mini` 는 이미지를 봅니다. 일부 텍스트 전용 모델은 못 봅니다.

??? failure "FileNotFoundError: sample.png"
    - 이미지가 **스크립트와 같은 폴더**에 있는지 확인
    - 파일명 대소문자·확장자 확인 (`sample.PNG` ≠ `sample.png`)
    - 터미널 현재 위치가 `ai901` 인지 확인

??? failure "TypeError: Object of type bytes is not JSON serializable"
    `.decode("utf-8")` 가 빠졌습니다. `b64encode()` 는 bytes를 돌려줍니다.

??? failure "URL 방식만 실패합니다"
    그 URL이 **공개 접근 가능**한지 브라우저 시크릿 창에서 확인하세요.
    로그인이 필요하거나 차단된 주소면 모델도 못 가져옵니다.

??? failure "요청이 너무 크다고 합니다"
    이미지가 큽니다. **1MB 이하**로 줄이거나 URL 방식을 쓰세요.
    base64는 33% 더 커진다는 걸 기억하세요.

??? failure "한글 설명이 아니라 영어로 답합니다"
    프롬프트에 `"한국어로 답해줘."` 를 명시하세요. 이미 있다면 더 앞쪽에 두세요.
