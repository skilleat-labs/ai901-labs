# Lab 11 — 음성 인식과 합성

!!! info "이 실습에서 하는 것"
    **글자를 소리로**(TTS), **소리를 글자로**(STT) 바꿉니다.
    Azure Speech는 표준(S0) 가격에 **월 50만 자 무료 한도**가 포함돼 비용 부담이 거의 없습니다.

**선행** [Lab 10](lab10-text-analysis.md) 완료 · **예상 시간** 25분 · **비용** 거의 0원 (무료 한도 내)

!!! note "“무료 한도”는 구독 단위로 공유됩니다"
    월 50만 뉴럴 문자는 **구독 전체에 주어지는 한도**입니다. 1인당이 아닙니다.
    이 실습은 1인당 **한두 문장**만 합성하므로 40명이 해도 한도 안에서 끝납니다.

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

## 1. Speech 키 준비 (5분)

1. [ai.azure.com](https://ai.azure.com) → 내 프로젝트 → **도구·지식** (또는 Tools)
2. **Speech** 를 찾아 연결하고, **키와 엔드포인트(또는 지역)** 를 확인합니다
3. `.env` 에 두 줄 추가합니다

```ini
SPEECH_KEY=여기에_Speech_키
SPEECH_REGION=리소스의_지역   # 예: eastus
```

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

!!! danger "★ `recognize_once` 는 한 문장만 듣습니다 — 시험 함정"
    | | 무엇 |
    |---|---|
    | `recognize_once_async()` | **한 번의 발화**만 인식하고 끝 |
    | 연속 인식 | 계속 듣습니다 (`start_continuous_recognition`) |

    "회의 전체를 받아적어야 한다"에 `recognize_once` 를 고르면 **오답**입니다.
    긴 녹음은 **일괄 처리(batch transcription)** 를 씁니다.

---

## 4. ✅ 체크포인트

!!! success "여기까지 됐으면 통과입니다"
    - [ ] `tts.py` 에서 **소리가 났다**
    - [ ] 목소리를 바꿔 봤다
    - [ ] `stt.py` 가 내 말을 **글자로** 받아적었다
    - [ ] `recognize_once` 와 연속 인식의 차이를 말할 수 있다

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
