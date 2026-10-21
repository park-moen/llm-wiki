# RAG 구축·비용·답변 범위와 평가

> Sources: Microsoft Azure Architecture Center, 공개일 미상; Microsoft Learn, 공개일 미상
> Raw: [RAG design and evaluation](../../raw/ai-engineering/microsoft-rag-design-and-evaluation.md); [RAG chunking phase](../../raw/ai-engineering/microsoft-rag-chunking-phase.md); [RAG and indexes](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)
> Updated: 2026-10-17

## 구축 흐름

RAG는 자료를 준비하는 처리와 사용자 질문에 답하는 처리를 나누어 설계한다. 준비 단계의 chunking(문서를 의미 있는 조각으로 나누기)·메타데이터·embedding·검색 설정이 실제 답변의 품질에 영향을 준다. [근거](../../raw/ai-engineering/microsoft-rag-design-and-evaluation.md)

| 자료 준비 | 질문 처리 |
|---|---|
| 자료 수집과 내용 확인 | 사용자 질문 수신 |
| 문서 분할과 제목·키워드 등 메타데이터 추가 | 관련 자료 검색 |
| vector 검색에 쓸 embedding 생성 | 검색 결과와 질문을 모델 입력으로 구성 |
| 검색 index에 저장 | 모델 호출과 답변 반환 |

표준 RAG는 미리 설계한 순서에 따라 검색과 모델 호출을 수행한다. 질문을 여러 검색으로 분해하거나 실행 중 자료원을 선택해야 할 때는 agentic RAG를 검토할 수 있다. 단순한 자료 기반 질의응답에 agent 구조부터 도입해야 한다는 뜻은 아니다. [근거](../../raw/ai-engineering/microsoft-rag-design-and-evaluation.md)

## 문서를 어떻게 나눌 것인가

문서 전체나 지나치게 큰 조각을 전달하면 비용과 입력 한도 문제가 생기고, 관련 없는 내용이 답변을 방해할 수 있다. 반대로 너무 작은 조각은 해석에 필요한 맥락을 잃는다. 원문은 고정 크기·문장 기반·의미 기반·문서 구조 활용 등의 접근을 비교하고, 문서별 품질과 처리 비용을 함께 고려하도록 설명한다. [근거](../../raw/ai-engineering/microsoft-rag-chunking-phase.md)

사주 자료에 적용한다면 해석 대상·조건·예외 설명이 함께 검색되는지 확인하는 방안을 생각할 수 있다. 예를 들어 조건 설명과 결론이 분리되면 다른 조건에 맞는 풀이를 가져올 수 있다. 이는 공식 문서의 chunking 원칙을 적용한 예시이며, 실제 프로젝트 자료를 확인해 정한 분할 규칙은 아니다.

고정된 정답 크기를 먼저 정하기보다 대표 자료와 질문으로 결과를 비교한다. 문서 추출 결과를 보관해 두면 분할 전략을 바꾸는 실험에서 앞선 처리의 반복 비용을 줄일 수 있다. 이미지·표 해석이나 모델을 이용한 전처리는 추가 비용을 검토해야 한다. [근거](../../raw/ai-engineering/microsoft-rag-chunking-phase.md)

## RAG가 비용을 줄이는 조건과 늘리는 조건

RAG를 추가하면 모델 호출만 하는 요청에 비해 검색 비용·지연과 embedding 비용이 생긴다. 검색한 자료도 모델 입력에 포함되므로 입력 token이 증가할 수 있다. 따라서 RAG 자체를 API 요금 할인이나 모델 경량화로 이해하면 안 된다. [근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

반면 문서 전체를 매번 전달하던 방식과 비교하면, 필요한 조각만 선별해 전달하는 것이 입력량을 줄일 수 있다. 비교 대상이 무엇인지에 따라 비용 판단이 달라진다. [근거](../../raw/ai-engineering/microsoft-rag-chunking-phase.md); [추가 비용 근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

| 비교 항목 | 비용에 미치는 영향 |
|---|---|
| 모델에 질문만 전달하던 방식에 RAG 추가 | 검색·embedding·참고 자료 입력 비용 추가 |
| 전체 자료 전달을 관련 조각 전달로 변경 | 입력량 절감 가능, 검색 관련 비용과 함께 비교 |
| 문서 분할·추출·이미지 설명 생성 | 자료 준비와 재처리 비용 발생 |
| 검색 자료를 과도하게 전달 | 입력 비용·지연·입력 한도 문제 가능 |

저가 모델과 RAG의 조합으로 충분한 품질을 달성할 수 있는지는 별도의 실험 가설이다. 작은 모델만 쓰면 된다는 결론은 원문에서 보장하지 않으며, 특정 모델이나 예상 요금도 이 문서에서 확정하지 않는다.

실무 비교에서는 같은 질문 묶음과 품질 기준을 사용해 모델 입력·출력량, 검색·embedding·저장 비용, 자료 갱신 비용, 지연과 재시도를 함께 기록하는 방안을 권한다. 이는 원문의 비용 항목과 단계별 평가 원칙을 조합한 검증 제안이다. [비용 근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md); [평가 근거](../../raw/ai-engineering/microsoft-rag-design-and-evaluation.md)

## 사주 자료만 검색하면 사주 질문에만 답하는가

검색 자료의 범위를 좁히는 것과 모델의 응답 범위를 제한하는 것은 다른 문제다. 공식 문서는 검색 자료를 입력에 추가하는 구조와 시스템 지침·애플리케이션 로직의 역할을 구분한다. 자료가 있어도 부정확한 답변이 생성될 수 있다는 한계도 설명한다. [근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

아래는 사주 챗봇 목적에 맞춰 제안한 처리 예시이며, 공식 문서가 제시한 완성된 구현이나 검증된 동작은 아니다.

1. 질문이 사주 관련 범위에 해당하는지 판단한다.
2. 범위 밖이면 사주 질문을 다루는 서비스라는 안내를 반환한다.
3. 범위 안이면 관련 자료와 사용자 사주 결과를 준비한다.
4. 근거가 부족하거나 결과가 없으면 추가 정보를 요청하거나 답변을 보류한다.
5. 근거가 충분하면 답변을 생성하고 자료 부합성과 응답 범위를 점검한다.

검색 유사도가 낮다는 이유만으로 사주 외 질문이라고 단정하면 안 된다. 사주 질문이지만 보유 자료에 답이 없을 수도 있기 때문이다. 질문 범위 판단과 근거 충분성 판단을 나눠 시험하자는 설계 제안이다.

사용자가 참고 자료나 질문에 넣은 지시로 응답 제한을 바꾸려는 경우도 시험한다. 검색 자료는 신뢰할 수 없는 입력으로 취급하고, 개인별 자료는 검색 단계에서 접근 권한을 적용해야 한다. [근거](../../raw/ai-engineering/microsoft-foundry-rag-and-indexes.md)

## 무엇을 평가해야 하는가

검색이 잘되는지와 최종 답변이 좋은지를 나누어 평가한다. 잘못된 검색 결과가 원인인지, 관련 자료를 전달했는데도 답변 생성이 실패한 것인지 구분해야 개선할 위치를 찾을 수 있다. [근거](../../raw/ai-engineering/microsoft-rag-design-and-evaluation.md)

| 평가 대상 | 확인할 질문 |
|---|---|
| 자료와 질문 준비 | 실제 자료를 대표하는 문서와 질문이 있는가? |
| 검색 | 질문에 필요한 내용이 검색되는가? |
| Groundedness(근거 부합성) | 답변이 전달된 자료를 근거로 작성됐는가? |
| Completeness(완전성) | 답변에 필요한 내용을 빠뜨리지 않았는가? |
| Relevancy(관련성) | 사용자가 묻는 내용에 답하는가? |

원문은 자료가 다루지 않는 질문도 평가에 포함하고, 설정과 결과를 기록하며, 각 단계와 전체 결과를 함께 확인하라고 권한다. [근거](../../raw/ai-engineering/microsoft-rag-design-and-evaluation.md)

사주 챗봇에서는 일반 사주 질문·조건이나 예외가 필요한 질문·보유 자료에 답이 없는 질문·사주 외 질문·응답 지침을 바꾸려는 질문을 같은 평가 묶음에 포함하는 방안을 제안한다. 이는 위 평가 원칙을 적용한 예시다. 답변이 자료에 부합한다는 평가와 사주 풀이 자체가 객관적으로 참이라는 판단도 구분한다.

## See Also

- [RAG의 원리와 기존 모델 API 활용](rag-fundamentals.md)
- [프로젝트의 AI 챗봇 MVP 출시 검토 — 대화 기반 archive](../projects/project-ai-mvp-discussion-2026-10-17.md)
