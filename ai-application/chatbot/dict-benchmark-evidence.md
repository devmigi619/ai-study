# 실측 요약 — 자연어 사전 가설 벤치

[자연어 사전 금지](./nl-dictionary-ban.md) · [닫힌/열린 분류](./retrieval-closed-vs-open-class.md) · [대화 이해는 LLM](./conversation-needs-llm.md) 의 경험적 근거. 제주 맛집 챗봇 PoC 측정.

환경: POI 999개, LLM-judge 블라인드(gpt-4o), rerank/answer(gpt-4o-mini), 임베딩 3종. 질의 22개(단발)+시나리오 7개(멀티턴).

## 1. 극성-함정 (결정론, LLM 무관)

질의 "혼자 먹기 좋은…", 리뷰만 다른 3 POI:

| 케이스 | pre-rerank 점수 |
|---|---|
| pos "혼자 좋았다" | 0.4625 |
| neg "혼자 별로" | 0.4625 (= pos, 극성 맹목) |
| oov "조용히"(사전밖, 진짜 적합) | 0.3350 (neg 보다 아래) |

부정 리뷰가 진짜 적합한 곳보다 위. 사전이 극성을 못 본다.

## 2. retrieve 매트릭스 (judge 평균 0-10)

arm: A=현행 사전 ranking, B=벡터 only, C=사전 recall 약혼합, D=사전 recall 전용.

| 임베딩 | A | B | C | D |
|---|---|---|---|---|
| 3-small | 5.55 | 4.23 | 4.23 | 4.23 |
| 3-large | 5.32 | 5.41 | 5.55 | 5.18 |
| bge-m3-korean | 5.77 | 5.32 | 5.18 | 5.09 |

버킷 핵심: **dict_hit(카테고리·지역 정확매칭)은 모든 임베딩에서 A 승**(bge 도 B=2.25). **situation·polarity·complex 는 강임베딩(large/bge)서 B/C 가 A 추월.** → 사전은 닫힌-분류에서만 가치, 열린의미·극성에선 해침. 임베딩이 좋을수록 사전 가치 축소(닫힌-분류 제외).

## 2b. LLM 플래너 arm (bge, A vs B vs P)

P = LLM 이 질의→{categories(택소노미 매핑), region, semantic, exclude}, 코드는 필터+벡터.

| arm | 평균 | 승(/22) |
|---|---|---|
| A 현행 사전 | 6.36 | 7 |
| B 벡터 only | 6.32 | 5 |
| **P 플래너** | **7.41** | **10** |

버킷: P 가 dict_hit 8.0(B 3.0), situation 9.0, polarity 7.33 으로 사전·벡터 둘 다 능가. 플래너 출력 예: D3"비오는날 따뜻한 국물"→categories=[국밥·해장국·찌개·전골·…], P2"웨이팅 없이"→exclude=[웨이팅].

## 3. 멀티턴 대화 (임베딩 무관)

DICT=사전 슬롯누적, LLM=히스토리 해소 질의재작성. 7 시나리오(정정·부정·모순·참조·화제복귀·공동탐색+filler).

| arm | 턴 평균 | 과제성공 |
|---|---|---|
| DICT | 5.76 | 1/7 |
| LLM | 9.14 | 7/7 |

단발 턴은 동률, 대화 깊어질수록 사전 붕괴(0~4점). 사전이 구조적으로 경쟁 불가.

## 한계

n 작음(방향성, 유의성 아님). retrieve 벤치 인덱스가 임베딩 텍스트에서 '태그:'줄 제거 → 닫힌-분류서 벡터 핸디캡(사전 우위 일부 과장). 멀티턴은 손-시나리오(공정하나 실트래픽 아님), DICT 엔 최선(슬롯union+벡터)·LLM 재작성기는 약모델(gpt-4o-mini)이라 격차는 보수적.

원본: `poc-chatbot/backend/bench/report.html` (+ `results_*.json`, `judge_*.json`, `run_matrix.sh`).

<!-- nav-backlinks -->

---
↑ [이 섹션 목차](./index.md) · [상위](../index.md) · [지식 베이스 홈](../../index.md)
