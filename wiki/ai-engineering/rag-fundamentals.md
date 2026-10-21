# RAG의 원리와 기존 모델 API 활용

> Sources: Microsoft Learn, 공개일 미상; AWS Prescriptive Guidance, 공개일 미상
> Raw: [RAG and indexes](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md); [RAG vs. fine-tuning](../../raw/ai-engineering/aws-rag-vs-fine-tuning.md)
> Updated: 2026-10-17

## 핵심 개념

RAG(Retrieval-Augmented Generation, 검색 증강 생성)는 질문에 관련된 자료를 찾아 모델 입력에 넣고, 그 자료를 근거로 답변을 생성하는 방식이다. 모델 자체의 이름이나 사전 학습 방법을 뜻하지 않는다. 기존 모델이 학습할 때 알지 못했던 내부 자료나 갱신된 정보를 답변에 활용할 수 있다. [근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

## 검색·자료 전달·답변 생성

| 단계 | 애플리케이션과 모델의 역할 |
|---|---|
| Retrieve | 질문에 관련된 내용을 index나 저장소에서 찾는다. |
| Augment | 질문과 검색한 자료를 하나의 모델 입력으로 구성한다. |
| Generate | 기존 모델이 입력받은 자료를 참고해 답변을 생성한다. |

검색은 애플리케이션이 수행하고 답변 생성은 모델이 수행한다. REST API나 SDK를 통해 검색과 모델 호출을 연결할 수 있다. 별도의 AI agent가 검색 도구를 선택하는 agentic RAG도 있지만, 모든 RAG가 agent 시스템을 요구하는 것은 아니다. [근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

## 왜 자체 모델 구축을 먼저 하지 않아도 되는가

모델의 문장 이해·생성 능력과 답변에 필요한 자료를 분리해서 사용할 수 있기 때문이다. 기존 모델에 관련 자료를 입력으로 전달하는 방식으로 시작할 수 있으며, 내부 지식을 모델에 추가 학습시키는 것이 RAG의 선행 조건은 아니다.

외부 모델 API를 사용한다면 모델을 실행하는 부분을 서비스로 이용하고, 애플리케이션은 검색·입력 구성·응답 처리를 연결한다. 따라서 API 연동은 모델 사용 경로이고 RAG는 자료를 찾아 활용하는 구조다. 둘은 대체 관계가 아니라 함께 사용하는 구성이다. 이 설명은 공식 문서의 검색·모델 호출 흐름을 구현 관점에서 풀어 쓴 것이다. [근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

## RAG와 fine-tuning의 차이

fine-tuning은 기존 모델을 추가 학습해 행동·문체·특정 작업의 성능을 조정하는 접근이다. RAG는 요청 시 관련 자료를 검색해 전달하는 접근이다. AWS는 보유 문서에 기반한 질의응답을 추가 학습 없이 구성할 수 있다고 설명하고, 해당 용도에서는 RAG부터 검토하도록 권한다. 두 방법을 함께 사용할 수도 있다. [AWS 근거](../../raw/ai-engineering/aws-rag-vs-fine-tuning.md); [Microsoft 근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

| 목적 | 구분해서 검토할 접근 |
|---|---|
| 보유 문서나 최신 자료를 근거로 답변 | RAG로 필요한 내용을 검색해 전달 |
| 모델의 문체·행동·작업 수행 방식 조정 | 지침 설계와 별도로 fine-tuning의 필요성 평가 |
| 자료 검색과 추가 학습 모두 필요 | RAG와 fine-tuning 조합 검토 |

특정 분야 자료만 검색한다고 모델의 다른 지식이 삭제되는 것은 아니다. 사주만 다루려는 목적은 자료 검색 외에 응답 범위를 관리하는 애플리케이션 설계로도 다뤄야 한다. 이는 RAG와 모델 행동 조정을 구분한 위 원문에서 도출한 설계상 해석이다.

## embedding과 검색 저장소

embedding은 내용을 숫자 표현으로 바꿔 vector 유사도 검색에 사용하는 것이다. 답변을 생성하는 모델과 검색용 embedding 모델은 역할이 다르다.

RAG의 검색 방식에는 키워드·의미 기반·vector·hybrid 검색이 있다. 따라서 RAG를 전용 vector DB 제품 하나와 동일하게 이해하면 안 된다. Azure AI Search는 이 원문이 설명하는 구현 선택지이며, 모든 RAG에 필수인 제품은 아니다. [근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

## 자료를 주면 항상 정확해지는가

검색된 내용이 부정확하거나 불완전하면 답변도 틀릴 수 있고, 자료가 있어도 모델이 근거 없는 답변을 생성할 수 있다. 자료 준비·검색 설정·응답 지침과 함께 결과를 평가해야 한다. 출처 표시도 가능하지만 실제 답변이 그 출처에 부합하는지 확인하는 과정이 필요하다. [근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

## See Also

- [RAG 구축·비용·답변 범위와 평가](rag-design-cost-and-evaluation.md)
- [프로젝트의 AI 챗봇 MVP 출시 검토 — 대화 기반 archive](../projects/project-ai-mvp-discussion-2026-10-17.md)
