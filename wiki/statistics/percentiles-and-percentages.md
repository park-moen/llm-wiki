# Percentile과 Percentage의 차이

> Sources: National Institute of Standards and Technology, Unknown; U.S. Bureau of Labor Statistics, Unknown
> Raw: [NIST Percentiles](../../raw/statistics/nist-percentiles.md); [BLS Distribution Statistics](../../raw/statistics/bls-distribution-statistics.md)
> Updated: 2026-09-21

## Overview

percentage(퍼센트)는 하나의 전체 중 얼마의 비율인지를 말하고, percentile(백분위수)은 값을 작은 순서로 정렬한 집단에서 특정 비율이 지나가는 **경계값**을 말한다. 따라서 `95%`와 `95th percentile`은 모두 95라는 숫자를 쓰지만, 질문과 답의 종류가 다르다. 전자는 “전체 중 얼마나 되는가”이고, 후자는 “정렬된 집단에서 하위 95%가 넘지 않는 값은 무엇인가”이다.

## Percentage: 전체를 100으로 본 비율

percentage는 기준이 되는 전체가 먼저 있어야 한다.

- 200개 요청 중 190개가 성공했다면 성공률은 `95%`입니다.
- 이 문장은 요청의 **비율**을 말할 뿐, 응답 시간이 얼마나 느렸는지는 말하지 않습니다.

즉 `95%`는 “100개 중 95개”라는 수량 관계입니다. 값들을 크기 순서로 배열할 필요가 없습니다.

## Percentile: 정렬된 값의 경계

percentile을 구하려면 같은 종류의 값을 모아 작은 순서부터 정렬합니다. NIST는 pth percentile을 데이터의 \(100p\)%가 그 값보다 작고, 나머지가 그 값보다 큰 경계값으로 설명합니다. 50th percentile은 median입니다. [NIST Percentiles](../../raw/statistics/nist-percentiles.md)

예를 들어 어떤 API 요청 100개의 응답 시간을 작은 순서로 놓았다고 가정합니다. `p95`가 `900ms`라면 다음처럼 읽습니다.

```text
응답 시간 100개를 정렬했을 때
95개는 900ms 이하
나머지 5개는 900ms보다 큼
```

여기서 `900ms`가 **95th percentile이라는 값**이고, `95`는 그 값 아래쪽에 놓인 데이터의 비율을 가리킵니다. `p95 = 900ms`는 “응답 시간의 95%가 900ms”라는 뜻도, “95%만 성공했다”는 뜻도 아닙니다.

## 95th percentile을 한 문장으로 읽는 법

`p95 = 900ms`는 다음 문장으로 읽으면 됩니다.

> 관측한 요청 중 약 95%는 900ms 이내에 끝났고, 가장 느린 약 5%는 900ms를 넘겼습니다.

따라서 p95는 평균 응답 시간이 아니라 느린 쪽 꼬리(tail)가 어디부터 시작하는지 보여 주는 경계입니다. 평균이 좋아도 일부 요청이 아주 느릴 수 있으므로, 응답 시간처럼 분포가 중요한 지표에는 p95를 함께 보는 이유가 된다. 이는 평균만으로 넓게 퍼진 분포를 충분히 설명하지 못할 수 있다는 BLS의 설명과 연결된다. [BLS Distribution Statistics](../../raw/statistics/bls-distribution-statistics.md)

## 자주 헷갈리는 표현

| 표현 | 올바른 해석 | 틀린 해석 |
| --- | --- | --- |
| 성공률 95% | 전체 요청의 95%가 성공 | 응답 시간의 95th percentile |
| p95 응답 시간 900ms | 약 95%의 응답 시간이 900ms 이하 | 평균 응답 시간이 900ms |
| 내 점수는 95th percentile | 비교 집단의 약 95%보다 높은 위치의 경계에 있음 | 시험에서 95점을 받음 |
| 50th percentile | median | 평균(mean) |

동점과 표본 크기 때문에 “이하”, “미만”, “약 95%”의 경계 표현은 계산 방식에 따라 조금 달라질 수 있다. percentile이 실제 관측값 사이에 놓이면 보간이 필요하며, NIST는 소프트웨어마다 여러 계산 방식이 쓰여 특히 작은 표본에서 결과가 완전히 같지 않을 수 있다고 설명한다. [NIST Percentiles](../../raw/statistics/nist-percentiles.md)

## 기억할 문장

**percentage는 비율이고, percentile은 그 비율의 데이터가 놓이는 위치를 가르는 값이다.**
