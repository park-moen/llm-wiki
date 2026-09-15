# uReview: Scalable, Trustworthy GenAI for Code Review at Uber

> Source: https://www.uber.com/us/en/blog/ureview/
> Collected: 2026-08-16
> Published: 2025-08-12

Uber의 uReview는 매주 약 65,000개의 diff 중 90% 이상을 분석한다. 사용자가 평가한 comment 중 75%가 유용하다고 표시됐고 게시된 comment의 65% 이상이 반영됐다.

단일 prompt로 review하지 않는다. 파일 선별, 전문 assistant별 comment 생성, confidence 평가, 중복 제거와 낮은 가치의 category 억제를 거친다. 개발자 feedback과 golden comments dataset을 이용해 model 조합을 평가한다.

AI는 인간 reviewer를 없애기보다 두 번째 reviewer로 배치되며, false positive와 hallucination을 관리하는 평가·filtering 체계가 핵심이다.

