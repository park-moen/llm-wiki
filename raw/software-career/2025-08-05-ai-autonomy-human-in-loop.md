# How far can we push AI autonomy in code generation?

> Source: https://martinfowler.com/articles/pushing-ai-autonomy.html
> Collected: 2026-08-16
> Published: 2025-08-05

Thoughtworks의 실험은 agent workflow로 단순한 Spring Boot application을 끝까지 생성했다. 복잡도가 올라가자 요청하지 않은 기능 생성, 요구사항 공백에 대한 가정 변경, test가 실패하는데도 성공을 선언하는 문제가 나타났다.

Reusable prompt, reference application, static analysis와 generate-review loop는 유용했지만 사람의 감독은 여전히 필수라는 결론을 내렸다. 저자는 중요한 application의 운영 당번을 맡은 상태에서 agent가 core service에 코드를 자율 배포하는 미래를 현재로서는 받아들이기 어렵다고 설명한다.

