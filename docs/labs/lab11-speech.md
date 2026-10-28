# Lab 11 — 음성 인식과 합성

!!! info "이 실습에서 하는 것"
    **글자를 소리로**(TTS), **소리를 글자로**(STT) 바꿉니다.
    Azure Speech는 표준(S0) 가격에 **월 50만 자 무료 한도**가 포함돼 비용 부담이 거의 없습니다.

**선행** [Lab 10](lab10-text-analysis.md) 완료 · **예상 시간** 27분 · **비용** 거의 0원 (무료 한도 내)

!!! note "“무료 한도”는 구독 단위로 공유됩니다"
    월 50만 뉴럴 문자는 **구독 전체에 주어지는 한도**입니다. 1인당이 아닙니다.
    이 실습은 1인당 **한두 문장**만 합성하므로 32명이 해도 한도 안에서 끝납니다.

    다만 긴 글을 반복해서 합성하면 한도를 넘어 과금될 수 있습니다.
    **짧은 문장으로** 확인하고 넘어가세요.

---

## 0. 방향을 먼저 정리합니다 (2분)

| | 영어 | 방향 | 오늘 |
|---|---|---|---|
| **음성 인식** | STT (Speech to Text) | 소리 → 글자 | 2절 |
| **음성 합성** | TTS (Text to Speech) | 글자 → 소리 | 1절 |

!!! note "★ 시험에서는 상황으로 물어봅니다"
    "전화 상담 내용을 분석하려 한다" → 먼저 **STT** 로 글자를 만들어야 합니다.
    "시각장애인용 안내 서비스" → **TTS**.

---

## 1. Speech 키 준비 (2분)

!!! success "★ Speech 리소스를 따로 만들지 마세요 — 지금 키 그대로입니다"
    Lab 01에서 만든 Foundry 리소스가 **Azure AI Services(멀티 서비스)** 라
    Speech 가 처음부터 들어 있습니다. 포털에서 뭔가 새로 만들 필요가 없습니다.
    (포털의 **Microsoft Foundry** = 예전 이름 **Azure AI Services** = 리소스 종류 `AIServices`, 같은 것입니다.)

    키는 Lab 02부터 쓰던 **`API_KEY` 그대로**, 지역은 리소스를 만든 **`eastus2`** 입니다.

`.env` 에 두 줄 추가합니다. 키는 위에 이미 있는 `API_KEY` 값을 **그대로 복사**하면 됩니다.

```ini
SPEECH_KEY=API_KEY 와 같은 값
SPEECH_REGION=eastus2
```

!!! danger "포털에서 "Speech" 를 검색하다 엉뚱한 걸 만들지 않도록"
    포털 검색에는 비슷해 보이는 항목이 여럿 나옵니다. 특히 **「검색 서비스」(Azure AI Search)** 는
    **이름이 비슷할 뿐 완전히 다른 서비스**이고, **표준 계층은 켜 두기만 해도 시간당 과금**됩니다.
    이 과정에서는 **Azure AI Search 를 쓰지 않습니다.** 실수로 만들었다면 지우세요.

패키지를 설치합니다.

```ini title="requirements.txt 에 추가"
azure-cognitiveservices-speech==1.53.0
```

```bash
pip install -r requirements.txt
```

---

## 2. TTS — 글자를 소리로 (8분)

먼저 쉬운 쪽부터 합니다. **스피커로 바로 들립니다.**

```python
import os
from dotenv import load_dotenv
import azure.cognitiveservices.speech as speechsdk

load_dotenv()

speech_config = speechsdk.SpeechConfig(
    subscription=os.getenv("SPEECH_KEY"),
    region=os.getenv("SPEECH_REGION"))

speech_config.speech_synthesis_voice_name = "ko-KR-SunHiNeural"

# 기본 스피커로 출력 — AudioConfig 없이도 됩니다
synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config)

result = synthesizer.speak_text_async(
    "안녕하세요. AI-901 실습에 오신 것을 환영합니다.").get()

print(result.reason)
```

```bash
python tts.py
```

!!! success "소리가 나면 성공입니다"
    헤드폰을 쓰시거나 볼륨을 줄여 주세요. 20명이 동시에 틀면 시끄럽습니다.

### 목소리를 바꿔 보세요

```python
speech_config.speech_synthesis_voice_name = "ko-KR-InJoonNeural"   # 남성
```

!!! note "★ SpeechConfig 와 AudioConfig 를 구분하세요"
    2026년 10월 합격자가 **헷갈렸던 지점**으로 지목한 짝입니다.

    | | 무엇을 정하나 | 필수인가 |
    |---|---|---|
    | **SpeechConfig** | **누구와 연결하나** — 키·지역·언어·목소리 | **항상 필요** |
    | **AudioConfig** | **어디로 넣고 빼나** — 마이크·스피커·파일 | **선택** |

    기본 스피커로 낼 때는 `AudioConfig` 없이도 됩니다.
    **파일로 저장**하거나 **파일을 입력**으로 쓸 때만 필요합니다.

---

## 3. STT — 소리를 글자로 (10분)

```python
import os
from dotenv import load_dotenv
import azure.cognitiveservices.speech as speechsdk

load_dotenv()

speech_config = speechsdk.SpeechConfig(
    subscription=os.getenv("SPEECH_KEY"),
    region=os.getenv("SPEECH_REGION"))
speech_config.speech_recognition_language = "ko-KR"

# 마이크 입력 (기본)
audio_config = speechsdk.audio.AudioConfig(use_default_microphone=True)

recognizer = speechsdk.SpeechRecognizer(
    speech_config=speech_config,
    audio_config=audio_config)

print("말씀하세요...")
result = recognizer.recognize_once_async().get()
print("인식:", result.text)
```

```bash
python stt.py
```

"말씀하세요"가 뜨면 **한 문장** 말해 보세요.

한 문장 말하고 나면 **프로그램이 그냥 끝납니다.** 두 번째 문장은 안 받아적습니다.
왜 그런지 다음 절에서 직접 확인합니다.

---

## 3-B. 연속 인식 — 계속 듣게 만듭니다 (5분)

!!! info "이 절을 따로 만든 이유"
    `recognize_once` 와 연속 인식의 차이는 **상황을 주고 고르게 하는 형태로 나옵니다.**
    설명만 읽고 넘어가면 시험장에서 헷갈립니다. **두 개를 다 돌려보면 안 헷갈립니다.**

`stt_cont.py` 를 만듭니다.

```python
import os, time
from dotenv import load_dotenv
import azure.cognitiveservices.speech as speechsdk

load_dotenv()

speech_config = speechsdk.SpeechConfig(
    subscription=os.getenv("SPEECH_KEY"),
    region=os.getenv("SPEECH_REGION"))
speech_config.speech_recognition_language = "ko-KR"

recognizer = speechsdk.SpeechRecognizer(
    speech_config=speech_config,
    audio_config=speechsdk.audio.AudioConfig(use_default_microphone=True))

# ① 한 문장이 확정될 때마다 불립니다
recognizer.recognized.connect(
    lambda evt: print("인식:", evt.result.text))

# ② 말하는 도중에도 계속 불립니다 (중간 결과)
recognizer.recognizing.connect(
    lambda evt: print("  ...", evt.result.text, end="\r"))

print("계속 듣습니다. 여러 문장 말해 보세요 (15초).")
recognizer.start_continuous_recognition()   # ③ 돌려놓고 바로 다음 줄로 갑니다
time.sleep(15)                              # ④ 그동안 계속 듣습니다
recognizer.stop_continuous_recognition()    # ⑤ 직접 멈춰야 끝납니다
print("\n끝.")
```

```bash
python stt_cont.py
```

**15초 동안 서너 문장을 끊어서** 말해 보세요. 문장마다 `인식:` 이 한 줄씩 찍힙니다.

### 무엇이 달라졌나

| | `recognize_once_async()` | `start_continuous_recognition()` |
|---|---|---|
| 몇 문장 | **한 번의 발화**만 | 멈출 때까지 **계속** |
| 결과를 받는 법 | `.get()` 의 **반환값** | `recognized` **이벤트 콜백** |
| 멈추는 법 | 알아서 끝남 | **`stop_...()` 을 직접 불러야** |
| 쓰는 곳 | 키오스크 · 음성 검색 · 짧은 명령 | 회의록 · 콜센터 전사 · 자막 |

!!! note "★ `start_continuous_recognition()` 은 **기다려 주지 않습니다**"
    이 줄은 "듣기를 시작해라"라고 **시켜만 놓고 바로 다음 줄로 넘어갑니다.**
    그래서 `time.sleep(15)` 로 프로그램을 **살려둬야** 합니다.
    이게 없으면 시작하자마자 프로그램이 끝나서 **아무것도 안 들립니다.**

!!! danger "★★ 시험은 세 갈래 중에 고르게 합니다"
    | 상황 | 답 |
    |---|---|
    | 키오스크에서 **짧은 문장** 하나 듣고 다음 단계로 | `recognize_once_async()` |
    | 마이크에서 **지속적으로** 인식 · 실시간 자막 | **연속 인식** (실시간 음성 텍스트 변환) |
    | 저장된 **녹음 파일 1,000건**을 분석 | **일괄 처리 (batch transcription)** |

    가장 흔한 함정은 **세 번째입니다.** 긴 녹음에 `recognize_once` 나 연속 인식을 고르면 오답입니다.
    문제에 "**실시간**"이 있으면 일괄 처리가 오답, "**대용량·저장된 파일**"이면 일괄 처리가 정답입니다.

---

## 4. ✅ 체크포인트

!!! success "여기까지 됐으면 통과입니다"
    - [ ] `tts.py` 에서 **소리가 났다**
    - [ ] 목소리를 바꿔 봤다
    - [ ] `stt.py` 가 내 말을 **글자로** 받아적었다
    - [ ] `stt_cont.py` 로 **여러 문장을 연속으로** 받아적었다
    - [ ] `recognize_once` 와 연속 인식의 차이를 말할 수 있다
    - [ ] 짧은 명령 / 실시간 연속 / 대용량 일괄 **세 갈래를 구분**할 수 있다

---

## 🔧 안 될 때

??? failure "Speech 키를 어디서 받는지 모르겠습니다"
    Foundry 프로젝트의 **도구·지식**에 Speech 연결이 없다면,
    Azure 포털에서 **Speech 서비스** 리소스를 따로 만들어도 됩니다
    (**`rg-ai901-<내 번호>` 안에**, 같은 지역으로). 키와 지역은 리소스 **개요**에 있습니다.

    헷갈리면 손을 드세요.

??? failure "소리가 안 납니다 (TTS)"
    1. `print(result.reason)` 이 무엇을 출력하는지 확인
    2. `Canceled` 면 키·지역이 틀렸을 가능성이 큽니다
    3. 시스템 볼륨과 출력 장치를 확인
    4. 파일로 저장해서 확인해 보세요

    ```python
    audio_config = speechsdk.audio.AudioOutputConfig(filename="out.wav")
    synthesizer = speechsdk.SpeechSynthesizer(
        speech_config=speech_config, audio_config=audio_config)
    ```

??? failure "마이크를 못 찾습니다 (STT)"
    - OS의 **마이크 권한**을 확인하세요 (macOS는 터미널에 권한 필요)
    - 실습실 PC에 마이크가 없으면 **음성 파일**로 대체합니다

    ```python
    audio_config = speechsdk.audio.AudioConfig(filename="sample.wav")
    ```

    TTS로 만든 `out.wav` 를 STT 입력으로 쓰면 **마이크 없이** 왕복을 확인할 수 있습니다.

??? failure "인식 결과가 비어 있습니다"
    - `result.reason` 을 출력해 보세요. `NoMatch` 면 소리가 너무 작았던 것입니다
    - 언어 설정이 `ko-KR` 인지 확인
    - 교실이 시끄러우면 인식률이 떨어집니다. 가까이서 또박또박 말해 보세요

??? failure "ModuleNotFoundError: azure.cognitiveservices.speech"
    ```bash
    pip install -r requirements.txt
    ```
    `(.venv)` 확인 먼저 하세요. `requirements.txt` 에
    `azure-cognitiveservices-speech==1.53.0` 줄이 있는지도 보세요.

---

## 💡 마이크가 없어도 되는 방법

TTS로 파일을 만들고 → 그 파일을 STT로 읽으면 **왕복 전체**를 확인할 수 있습니다.

```text
"안녕하세요" → [TTS] → out.wav → [STT] → "안녕하세요"
```

실습실 환경이 제한적이면 이 방법을 쓰세요.
