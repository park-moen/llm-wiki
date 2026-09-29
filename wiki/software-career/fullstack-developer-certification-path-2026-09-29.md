# 풀스택 개발자를 위한 SQL·DB·AWS 자격증 선택

> Sources: [AI 중심 실무 환경의 Full-stack 개발자 6개월 학습 로드맵](ai-native-fullstack-learning-roadmap-2026-08-23.md)
> Archived: 2026-09-29

## Overview

전문대 졸업·비전공 조건에서 React·Next.js와 Kotlin·Spring Boot를 다루는 개발자가 SQL·DB·클라우드 자격증을 선택할 때의 판단을 정리한 대화 기록이다. 범용 SQL 기초에는 SQLD, 실제 AWS 애플리케이션 개발·배포 경험을 검증하는 데는 AWS Certified Developer – Associate가 우선 후보였다. 상위 자격증은 자동으로 이어서 취득하기보다 담당 업무와 지원 직무에 맞춰 선택한다. 채용 효과의 크기를 측정한 자료는 확인하지 않았으므로 아래 우선순위는 시험 범위와 현재 학습 목표를 비교한 판단이다.

## 자격증별 역할과 응시 조건

| 자격증 | 성격과 적용 범위 | 전문대·비전공 조건에서의 판단 |
| --- | --- | --- |
| SQLD | 한국데이터산업진흥원의 국가공인 민간자격. SQL과 데이터 모델링 기초 | 응시 자격 제한 없음. [시행기관 안내](https://www.dataq.or.kr/www/dataq_brochure_2022.pdf) |
| SQLP | SQL 고급 활용·튜닝을 평가하는 국가공인 민간자격 | SQLD 취득이 응시 자격 경로 중 하나다. 복잡한 SQL의 성능 문제를 다룰 때 우선순위가 높아진다. [시행기관 안내](https://www.dataq.or.kr/www/dataq_brochure_2022.pdf) |
| 정보처리산업기사·정보처리기사 | 개발 전반을 다루는 국가기술자격 | Q-net은 두 종목의 관련 학과를 모든 학과로 안내한다. 다만 기사 등급에는 별도 학력·경력 요건이 있으므로 개인의 응시 가능 여부는 Q-net 자가진단으로 확인한다. [산업기사](https://www.q-net.or.kr/crf005.do?id=crf00503s02&jmCd=2290&jmInfoDivCcd=B0), [기사](https://www.q-net.or.kr/crf005.do?id=crf00503&jmCd=1320&tabGbn=1), [응시 자격](https://www.q-net.or.kr/crf006.do?gId=57&gSite=L&gradeType=30&id=crf00603&tabIdx=2) |
| Oracle OCA·OCP·OCM | Oracle 제품에 대한 기업 인증. Associate·Professional·Master 수준 | Oracle DB 운영·튜닝 직무에서 관련성이 높다. 시험별 선행 조건이 다르므로 공통된 OCA → OCP → OCM 필수 순서로 이해하지 않는다. [Oracle 인증](https://www.oracle.com/kr/education/certification/) |

Oracle Database Administration 2019 OCP는 OCA 선취득 없이 Administration I·II 시험으로 취득하도록 안내한다. 반면 Oracle Database 19c OCM은 선행 OCP와 지정된 고급 교육을 요구한다. Oracle의 현재 인증 목록에는 Oracle AI Database SQL Associate와 Administration Associate·Professional도 있으므로 준비할 때 제품 버전과 시험명을 확인해야 한다. [2019 OCP 공식 FAQ](https://www.oracle.com/a/ocom/docs/dc/ww-ou-5297-database2019-faq-3.pdf), [19c OCM 응시 안내](https://www.oracle.com/cn/education/database-19c-oracle-certified-master-exam/), [Oracle 인증 목록](https://www.oracle.com/kr/education/certification/)

## 풀스택 경력에 맞춘 순서

**AWS Certified Developer – Associate는 하나의 자격증 이름**이다. 별도로 말한 “Associate”가 AWS Certified Solutions Architect – Associate를 뜻한다면 두 자격증은 개발·배포와 아키텍처 설계라는 서로 다른 범위를 다룬다. [Developer – Associate](https://aws.amazon.com/certification/certified-developer-associate/), [Solutions Architect – Associate 시험 범위](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03.html)

1. SQL 기초와 데이터 모델링을 보강할 필요가 있으면 SQLD를 준비한다.
2. 실제 업무나 목표 직무에서 AWS를 사용한다면 AWS Certified Developer – Associate를 준비한다. 시험 공부를 애플리케이션 배포·운영 경험과 연결한다.
3. 이후 서비스의 보안·가용성·비용 설계를 맡는다면 Solutions Architect – Associate를, 실행 계획과 SQL 튜닝을 맡는다면 SQLP를 선택한다.
4. Oracle OCP는 Oracle DB가 주력인 직무에서, OCM은 Oracle DB 운영 경험과 선행 요건을 갖춘 뒤 고려한다.

SQLD와 AWS Developer – Associate를 함께 학습하는 것은 가능하지만, 시험 준비량이 커지면 취득은 차례로 진행한다. 지원 회사가 AWS를 사용하지 않는다면 AWS 시험 준비를 우선할 근거가 약하다. 한국의 개발자 이직에서 정보처리기사도 고려할 수 있지만, 응시 자격과 목표 공고의 요구 사항을 먼저 확인한다. 이 순서는 자격증 자체가 채용 성과를 보장한다는 뜻이 아니다. [Full-stack 학습 로드맵](ai-native-fullstack-learning-roadmap-2026-08-23.md)은 화면 → API → DB → 테스트 → 배포 흐름을 직접 설명하고 검증하는 역량을 우선한다.

### 공부를 시작하는 방법

먼저 지원하려는 직무의 기술 범위와 현재 업무를 대조한다. SQL을 직접 작성·검토하는지, AWS에 배포하거나 운영 문제를 살피는지 확인한 뒤 **당장 쓰는 쪽을 주 학습 과목**으로 정한다. 두 시험을 병행한다면 나머지 한쪽은 개념 복습과 짧은 실습으로 유지하고, 첫 시험을 치른 뒤 집중도를 바꾼다. 시험 날짜부터 모두 확정할 필요는 없다.

학습 사례는 작은 기능 하나로 통일한다. 예를 들어 React·Next.js 화면에서 목록을 요청하고, Kotlin·Spring Boot API가 관계형 DB를 조회해 응답하는 흐름을 고른다. 이미 다룬 업무 기능을 활용할 수 있으면 그 흐름을 따라가고, 그렇지 않으면 별도 연습 기능을 만든다. 화면 → API → DB → 테스트 → 배포의 각 지점에서 요청과 실패가 어떻게 처리되는지 기록한다. [Full-stack 학습 로드맵](ai-native-fullstack-learning-roadmap-2026-08-23.md)

| 단계 | 공부할 내용 | 다음 단계로 넘어갈 기준 |
| --- | --- | --- |
| SQLD 기초 | 데이터 모델링과 SQL 기본·활용 범위를 공식 시험 안내로 확인한다. 연습 기능의 테이블 관계를 그리고 조회·집계 쿼리를 직접 작성한다. | 쿼리 결과가 왜 그렇게 나오는지, 조인 조건과 모델의 관계를 설명하고 테스트 데이터로 확인할 수 있다. [SQLD 과목 안내](https://www.dataq.or.kr/www/dataq_brochure_2022.pdf) |
| AWS Developer – Associate 기초 | 응시할 시험 버전의 공식 가이드를 기준으로 AWS 서비스 개발, 보안, 배포, 문제 해결을 공부한다. 연습 기능을 배포하고 권한·설정·로그·실패 복구 경로를 확인한다. | 배포된 기능을 호출하고, 권한 오류나 배포 실패가 났을 때 확인할 로그와 설정을 설명할 수 있다. [현재 DVA-C02 시험 가이드](https://docs.aws.amazon.com/aws-certification/latest/developer-associate-02/developer-associate-02.html) |
| 시험 직전 | 공식 시험 범위와 예시 문제로 빈 곳을 찾고, 틀린 항목을 같은 기능의 코드·쿼리·배포 설정에서 다시 확인한다. | 문제의 정답뿐 아니라 다른 선택지가 왜 맞지 않는지 설명할 수 있다. [AWS 공식 준비 자료](https://aws.amazon.com/certification/certification-prep/) |

### 첫 자격증을 취득한 뒤

- **Solutions Architect – Associate:** 실제로 아키텍처 선택을 해야 할 때 진행한다. 같은 기능의 구성도를 그린 뒤 보안, 장애 대응, 성능, 비용 요구가 바뀌었을 때 어떤 구성을 선택할지 비교한다. 이 네 영역은 시험 가이드의 핵심 범위다. [Solutions Architect – Associate 시험 가이드](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03.html)
- **SQLP:** 복잡한 쿼리의 성능 문제를 다루기 시작할 때 진행한다. 실제 데이터 규모를 가정해 실행 계획과 인덱스·조인 방식을 비교하고, 변경 전후의 결과와 성능을 검증한다. SQLP는 SQL 고급 활용·튜닝을 평가한다. [SQLP 과목 안내](https://www.dataq.or.kr/www/dataq_brochure_2022.pdf)

두 상위 자격증을 모두 목표로 정하기 전에, 최근 맡은 업무에서 어느 쪽 판단을 더 자주 했는지 확인한다. 자격증을 취득하지 않더라도 공부 과정에서 만든 쿼리 검증 기록, 배포·장애 분석 기록, 설계 선택의 근거는 이직 때 설명할 실무 사례가 된다.

## 유효 기간과 유지 방법

| 자격증 | 유효 기간 | 유지 방법 |
| --- | --- | --- |
| AWS Certified Developer – Associate | 취득 후 3년 | 만료 전에 최신 버전의 같은 시험을 다시 통과하거나 AWS가 인정하는 상위 시험으로 갱신한다. 현재 AWS는 조건을 충족하면 Skill Builder 활동으로 1년 연장하는 방법도 안내한다. [AWS 갱신 안내](https://aws.amazon.com/certification/recertification/) |
| SQLD·SQLP | 취득 후 2년 | 시행기관 안내에 따르면 보수교육 이수 후 영구 자격으로 전환된다. [데이터자격시험 안내](https://www.dataq.or.kr/www/dataq_brochure_2022.pdf) |

AWS Solutions Architect – Associate를 추가 취득하는 것만으로 Developer – Associate가 갱신되지는 않는다. AWS는 두 자격증의 갱신 경로를 각각 안내한다. SQLD·SQLP의 유효 기간 근거로 사용한 시행기관 자료는 2022년 발행본이므로 실제 응시·보수교육 신청 시 최신 공지를 다시 확인한다. [AWS 갱신 경로](https://aws.amazon.com/certification/recertification/)

## 시점에 따른 확인 사항

2026년 9월 29일 기준 AWS는 Developer – Associate의 새 시험인 DVA-C03 등록을 2026년 10월 27일부터 시작한다고 안내한다. 시험 범위와 갱신 정책은 바뀔 수 있으므로 응시 직전에 공식 페이지를 확인한다. [AWS 시험 안내](https://aws.amazon.com/certification/certified-developer-associate/)

## See Also

- [AI 중심 실무 환경의 Full-stack 개발자 6개월 학습 로드맵](ai-native-fullstack-learning-roadmap-2026-08-23.md)
