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

## 1. 준비 — 이미지 한 장 (3분)

`ai901` 폴더에 이미지를 하나 넣습니다.

- 휴대폰 사진, 스크린샷, 아무거나 괜찮습니다
- 파일명을 **`sample.png`** (또는 `.jpg`) 로 저장하세요
- **1MB 이하**를 권합니다. 크면 느리고 토큰이 많이 듭니다

!!! tip "글자가 들어간 이미지를 추천합니다"
    영수증, 간판, 표지판 같은 걸 쓰면 모델이 글자까지 읽는 걸 볼 수 있습니다.
    Lab 15(정보 추출)와도 자연스럽게 이어집니다.

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
    `gpt-4o-mini` 는 이미지를 봅니다. 일부 텍스트 전용 모델은 못 봅니다.

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
