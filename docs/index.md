# AI-901 실습 랩

**Microsoft Foundry로 직접 만들어 보는** AI-901 4일 hands-on 가이드입니다.
강의에서 개념을 듣고, 바로 여기서 손으로 만듭니다.

!!! info "대상 및 환경"
    - **대상**: AI-901 집체 과정 수강생 · AI/Azure 사전 지식 불필요
    - **언어**: Python 3.10+ (코드는 전부 10~20줄 수준입니다)
    - 실습 비용은 **강사의 교육용 구독**에서 나갑니다 (1인 몇십 센트 수준)
    - **신용카드 등록은 어느 단계에도 없습니다**

!!! warning "Azure 계정 — 강사가 배부합니다"
    **가입 절차가 없습니다.** 1일차에 계정과 전용 리소스 그룹을 배부받습니다.

    ```text
    강사의 교육용 구독
    ├─ rg-ai901-01   ← 1번 수강생의 자리
    ├─ rg-ai901-02   ← 2번 수강생의 자리
    └─ ...
    ```

    | 받아서 시작하는 것 | 직접 만드는 것 |
    |---|---|
    | 구독 · 예산 알림 | Foundry 리소스·프로젝트 (Lab 01) |
    | 내 계정 `ai901-<번호>` | 모델 배포 (Lab 02) |
    | 내 리소스 그룹 `rg-ai901-<번호>` | 에이전트 (Lab 08) |

    **리소스 그룹 안의 모든 것은 직접 만듭니다.** 그 절차 자체가 시험 범위입니다.
    사정상 본인 구독으로 진행하는 경우는 [Lab 00](labs/lab00-subscription.md) 맨 아래를 보세요.

## 랩 목록

| 랩 | 일차 | 무엇을 만드나 | 분 |
|---|---|---|---|
| [Lab 00](labs/lab00-subscription.md) | 1일차 | 내 실습 환경 확인 + 권한 경계 | 30 |
| [Lab 01](labs/lab01-foundry-project.md) | 1일차 | Foundry 리소스·프로젝트 생성 | 40 |
| [Lab 02](labs/lab02-first-call.md) | 1일차 | 키·Python 환경·첫 연결 | 45 |
| [Lab 03](labs/lab03-deploy-model.md) | 2일차 | 모델 배포 — 유형·버전·TPM·가드레일 | 35 |
| [Lab 04](labs/lab04-tokens.md) | 2일차 | 첫 호출 확장과 토큰 사용량 | 30 |
| [Lab 05](labs/lab05-chat-memory.md) | 2일차 | 대화를 기억하는 챗봇 (messages) | 35 |
| [Lab 06](labs/lab06-parameters.md) | 2일차 | 매개변수 실험 (온도·최대 토큰) | 20 |
| [Lab 07](labs/lab07-system-grounding.md) | 2일차 | 시스템 메시지와 근거 기반 답변 | 25 |
| [Lab 08](labs/lab08-agent-portal.md) | 3일차 | 포털에서 에이전트 + 지식 붙이기 | 40 |
| [Lab 09](labs/lab09-agent-code.md) | 3일차 | 코드에서 에이전트 호출 (Azure CLI 설치 + ID 인증) | 36~46 |
| [Lab 10](labs/lab10-text-analysis.md) | 3일차 | 텍스트 분석 — 범용 모델 vs 전용 서비스 | 30 |
| [Lab 11](labs/lab11-speech.md) | 3일차 | 음성 인식·합성 (STT·TTS) | 25 |
| [Lab 12](labs/lab12-voice-live-demo.md) | 3일차 | Voice Live 실시간 — **강사 데모** | 15 |
| [Lab 13](labs/lab13-multimodal-image.md) | 4일차 | 멀티모달 이미지 (URL·base64) | 36 |
| [Lab 14](labs/lab14-image-generation.md) | 4일차 | 이미지 생성 ⚠️ 1인 1장 | 20 |
| [Lab 15](labs/lab15-content-understanding.md) | 4일차 | Content Understanding 영수증 추출 | 40 |
| [Lab 16](labs/lab16-cleanup.md) | 4일차 | 리소스 정리 | 12 |

**17개 랩(Lab 00~16) 전체가 준비되어 있습니다.**

## 시작하기

!!! danger "수업 전에 — 노트북에 Python 을 설치해 주세요"
    **[수업 전 설치](install.md)** 를 보고 미리 해두시면 1일차가 훨씬 수월합니다. 10분이면 됩니다.
    못 하셨어도 괜찮습니다 — 당일 도와드리고, 브라우저로 하는 방법도 있습니다.

먼저 [시작하기](00-setup.md) 에서 준비물을 확인하고,
[Lab 00](labs/lab00-subscription.md) 부터 순서대로 진행하세요.
**각 랩은 앞 랩 위에 쌓입니다.** 건너뛰면 뒤에서 막힙니다.

## 이 과정이 자격증과 이어지는 지점

여기서 만드는 것들이 곧 **AI-901 시험 범위**입니다.

- 시험의 **55~60%** 가 "Microsoft Foundry로 AI 솔루션 구현"입니다
- 2026년 응시 후기에 따르면 **Python SDK 코드의 빈칸을 채우는 문제**가 나옵니다
- 서비스 이름만 외워서는 풀리지 않습니다. **직접 쳐본 사람이 붙습니다**

!!! tip "코드는 꼭 직접 쳐보세요"
    복사만 하면 시험장에서 기억나지 않습니다.
    특히 `client = OpenAI(...)` 와 `client.responses.create(...)` 의 차이는
    시험에 그대로 나옵니다.
