# How AI will change software engineering – with Martin Fowler

> Source: https://www.youtube.com/watch?v=CQmI4XKTa0U
> Collected: 2026-08-16
> Published: 2025-11-19

The Pragmatic Engineer의 Gergely Orosz가 Thoughtworks Chief Scientist Martin Fowler를 인터뷰한 episode다. 공식 episode page는 전체 transcript와 참고 자료를 제공한다.

- Official transcript: https://newsletter.pragmaticengineer.com/p/martin-fowler
- Duration: 1:48:53
- YouTube captions: English, auto-generated

## Official chapter outline

- 00:00 Intro
- 01:50 How Martin got into software engineering
- 07:48 Joining Thoughtworks
- 10:07 The Thoughtworks Technology Radar
- 16:45 From Assembly to high-level languages
- 25:08 Non-determinism
- 33:38 Vibe coding
- 39:22 StackOverflow vs. coding with AI
- 43:25 Importance of testing with LLMs
- 50:45 LLMs for enterprise software
- 56:38 Why Martin wrote Refactoring
- 1:02:15 Why refactoring is so relevant today
- 1:06:10 Using LLMs with deterministic tools
- 1:07:36 Patterns of Enterprise Application Architecture
- 1:18:26 The Agile Manifesto
- 1:28:35 How Martin learns about AI
- 1:34:58 Advice for junior engineers
- 1:37:44 The state of the tech industry today
- 1:42:40 Rapid fire round

## Short transcript evidence

> “If you're not looking at the output, you're not learning.”

> “Don't trust, but do verify.”

> “Find some good senior engineers who will mentor you.”

## Source observations

이하는 transcript를 대체하는 인용문이 아니라, 위 chapter를 다시 찾기 위한 내용 색인이다.

- High-level language는 hardware 세부사항에서 벗어나 abstraction을 만들 수 있게 했지만 deterministic 성격은 유지했다. LLM의 더 큰 변화는 같은 입력이 항상 같은 출력을 보장하지 않는 non-determinism이다.
- Structural engineering의 tolerance처럼, AI를 사용하는 software engineering에도 최악의 결과를 감안한 여유와 검증 장치가 필요하다.
- Vibe coding은 결과 code를 이해하지 않는다는 원래 의미로 사용한다. Prototype, exploration과 disposable tool에는 유용하지만 장기 유지 software에는 learning loop와 수정 가능성을 잃게 한다.
- LLM은 unfamiliar environment 탐색, 초기 project skeleton, legacy system 이해에 유용하다.
- AI 변경은 생산적이지만 신뢰하기 어려운 collaborator의 작은 pull request처럼 다뤄야 한다. 작은 slice, review와 test가 필요하다.
- 큰 specification을 먼저 완성하는 방식은 waterfall 문제를 되살릴 수 있다. 최소 specification, 구현, test, production feedback을 짧게 반복하는 편이 중요하다.
- Refactoring은 동작을 보존하는 작은 변경을 조합하는 practice다. LLM 단독보다 IDE refactoring, static analysis와 같은 deterministic tool과 결합하는 방향이 유망하다.
- Enterprise software는 regulation, legacy system, business knowledge와 risk tolerance 때문에 startup과 다른 판단이 필요하다.
- Agile의 핵심은 ceremony가 아니라 작은 increment와 빠른 feedback loop다. AI는 한 번에 더 큰 변경을 만드는 것보다 cycle time을 줄이는 데 쓰는 편이 낫다.
- 좋은 정보원은 확신만 내세우지 않고 context, trade-off와 불확실성을 드러낸다.
- Junior는 AI를 실험해야 하지만 결과 품질을 판단할 경험이 부족하므로 좋은 senior mentor와 source를 의도적으로 찾아야 한다.
- Software developer의 핵심은 code 입력 속도만이 아니라 무엇을 만들지 이해하고 사용자와 소통하는 능력이다.

