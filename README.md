# BRANDPILOT

생성형 AI를 활용해 기업 진단, 네이밍, 콘셉트, 스토리, 로고를 단계별로 만드는 브랜딩 컨설팅 플랫폼입니다.

| 항목 | 내용 |
| --- | --- |
| 기간 | 2025.12 - 2026.02 |
| 팀 구성 | 6인 팀, AI 개발자 2인 공동 개발 |
| 개인 담당 | AI 측 FastAPI 연동, 요청과 응답 형식 조율 |
| 공동 작업 | 생성 Node, State와 Context, Prompt, 이미지 전달 로직 |
| 팀 결과 | 서비스 연동과 배포, AIVLE Big Project Practical상 |

이 저장소에는 프로젝트 설명을 정리했습니다. 실제 AI 구현은 공개된 [팀 AI 저장소](https://github.com/AivleSchool-BigProject/branding-ai)에서 확인할 수 있습니다.

## 문제와 제약

각 단계에서 사용자가 선택한 결과를 다음 생성에 반영해야 했습니다. AI 모델이 만든 결과는 화면에 표시할 정보, 저장할 후보, 다음 단계에서 사용할 상세 Context로 구분해 전달해야 했습니다.

연동 과정에서는 Backend와 AI가 같은 데이터를 다른 필드명과 계층으로 정의해 요청과 응답 형식이 맞지 않는 문제가 있었습니다. 이미지 모델의 Base64 응답도 서비스에서 사용할 전달 형식으로 바꿔야 했습니다.

## 단계별 실행 구조

진단 → 네이밍 → 콘셉트 → 스토리 → 로고를 단계별 Node로 구성했습니다. 하나의 요청으로 전체 단계를 연속 실행하는 구조가 아니라 요청의 `current_step`에 해당하는 Node 하나를 실행하고 종료합니다.

백엔드는 사용자가 선택한 이전 결과와 상세 Context를 저장한 뒤 다음 요청에 포함합니다. AI 서버는 이 정보를 새 State에 넣어 현재 단계 생성에 사용합니다.

## 담당 역할과 협업

AI 코드베이스는 AI 개발자 2인이 논의하며 공동으로 설계하고 구현했습니다. 그중 AI 측 FastAPI 연동을 맡아 백엔드 담당자와 JSON의 필드명, 계층과 자료형을 맞췄습니다.

- 공통 요청과 응답 형식을 정하고 이전 입력 형식도 처리하도록 구성했습니다.
- 문자열로 전달된 JSON을 객체로 변환해 각 생성 Node에 같은 구조로 전달했습니다.
- 화면 표시와 저장용 결과, 다음 단계에 필요한 Context를 구분해 응답했습니다.
- Swagger로 진단부터 로고까지 호출하고 백엔드 담당자와 DB 저장까지 확인했습니다.

## 이미지 전달

후보 이미지 전달 흐름을 공동 구현했습니다.

```text
Gemini Base64
  → AI 서버에서 PNG로 변환
  → Cloudinary 업로드
  → 후보 이미지 URL을 FastAPI로 응답

사용자가 최종 후보 선택
  → 백엔드에서 이미지 다운로드
  → S3 업로드
  → 최종 S3 URL을 DB에 저장
```

후보 전달에는 Cloudinary를, 최종 선택본 보관에는 S3를 사용합니다.

## 모델과 Prompt 변경

구조화 출력과 로고 내 브랜드명 텍스트 표현 문제를 다루며 GPT-4o에서 GPT-5.1로, DALL-E 3에서 Gemini 3 Pro Image로 전환하는 작업에 참여했습니다. Prompt 구조화와 모델 관련 변경은 AI 공동 작업자와 논의하며 수행했습니다.

단계별 API 호출과 백엔드 DB 저장까지 서비스 연동을 확인했습니다.

## 구현 코드

| 확인할 내용 | 팀 저장소의 파일 |
| --- | --- |
| 현재 단계의 Node 선택과 종료 | [graph.py](https://github.com/AivleSchool-BigProject/branding-ai/blob/main/langgraph_system/graph.py) |
| FastAPI 요청 처리와 State 구성 | [brand.py](https://github.com/AivleSchool-BigProject/branding-ai/blob/main/api/routers/brand.py) |
| 입력 형식 처리 | [request.py](https://github.com/AivleSchool-BigProject/branding-ai/blob/main/api/schemas/request.py) |
| 화면용 결과와 다음 단계 정보의 분리 | [response.py](https://github.com/AivleSchool-BigProject/branding-ai/blob/main/api/schemas/response.py) |
| 이미지 생성과 Cloudinary 후보 전달 | [logo_node.py](https://github.com/AivleSchool-BigProject/branding-ai/blob/main/langgraph_system/nodes/logo_node.py) |

사용 기술: Python, LangGraph, FastAPI, GPT-5.1, Gemini 3 Pro Image, Cloudinary
