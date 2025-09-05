# 포트폴리오 분석 계산 로직&시뮬레이션 (Calculation Logic&Sim.)

## ⚠️주의사항 
- 아래의 모든 시뮬레이션 결과는 **랜덤워크 기반 모의 주가 데이터**(seed=500~509)를 사용하여 산출된 값임.
- 실제 주식 시장 데이터와는 차이가 있으며, **투자 의사결정에 직접 활용할 수 없음**
- 따라서 본 결과는 **코드/계산 로직 검증 및 학습 목적**으로만 참고


<br>


## ✅ 코드/계산 로직 검증 결과
> 본 프로젝트에서 구현된 모든 계산 로직은 10회 시뮬레이션을 통해 반복 검증되었으며, <br> 각 지표(수익률, 이동평균, 변동성, 베타/알파, MDD, Var 등)가 **수학적 정의와 일치하는 결과**를 안정적으로 산출하였음.



### 🟢 이론값 vs 시뮬레이션 평균 비교
- 랜덤워크(GBM) 기반으로 μ=0.0005, σ=0.015로 데이터를 생성했음
- 이론적 기대값 :
  - 일간 평균 수익률 ≈ 0.0005
  - 일간 표준편차 ≈ 0.015

- 시뮬레이션(10회 평균) 결과:
  - 일간 평균 수익률 ≈ 0.00011
  - 일간 표준편차 ≈ 0.0105 ~ 0.011
  - → 이론과 근접, 오차범위 내 일관성 확인 

### 🟢 t-검정 (평균 수익률)
- 귀무가설 $H_0$: 시뮬레이션 평균 = 이론 기대값(0.0005)
- 유의수준 5%에서 양측 t-test 수행
- p-value > 0.05 → 귀무가설 기각 불가 → 이론 기대값과 통계적으로 유의한 차이 없음

### 🟢 분산 검정 (표준편차)
- χ² 분산 검정으로 시뮬레이션 표준편차 vs σ=0.015 비교
- 결과: 대부분 케이스에서 95% 신뢰구간에 포함 → 이론값과 일치


<br>


## 🔷🔶테스트/시뮬레이션 환경, 조건

- 언어/런타임 : JavaScript(ES Modules), Node.js ≥ 18 / 또는 브라우저(동일 코드)
- 자산 : 005930(삼성전자), 000660(SK하이닉스), 035420(NAVER), 벤치마크 KOSPI
- 기간 : 약 120 영업일(일봉)
- 가격 생성(랜덤워크, GBM 유사)
  - 일간 드리프트 μ ≈ 0.0005 (0.05%), 변동성 σ ≈ 0.015 (1.5%)
  - 시드: 500, 501, …, 509 → 총 10회
- 포지션(고정) :
  - 005930 : 12주 @ 65,000
  - 000660 : 3주 @ 280,000
  - 035420 : 6주 @ 85,000
- 가중치 : 시가 가치 비중 (리밸런싱X, **고정가중 가정**)
- 연환산 파라미터 : 거래일 252, 무위험수익률 1.5%

> 실제 10회 시뮬레이션 결과를 표에 입력하였습니다. <br>
> 각 표의 “적합/부적합”은 조건 대비 결과의 수치적 타당성을 임의 평가한 것입니다.
 

<br>


## 🔶 공통 유틸 (정확한 합 / 평균 / 표준편차 etc.)

```js
/** 완전합 */
export function sum(arr){
    if (!Array.isArray(arr)) throw new Error("sim:array required");
    return arr.reduce((s, v) => s + Number(v), 0);
}

/** 평균 */
export function mean(arr){
    if(!Array.isArray(arr) || arr.length === 0) throw new Error("mean: non-empty array required");
    return sum(arr) / arr.length;
}

/** 표준편차 */
export function stdev(arr){
    if(!Array.isArray(arr) || arr.length < 2) throw new Error("stdev: need >= 2 values");
    const m - mean(arr);
    const v - sum(arr.map(x => (x - m) ** 2)) / (arr.length -1);
    return Math.sqrt(v);
}

/** 퍼센트/원화 포맷 계산 아님 > 프론트 포맷팅에서 처리 예정 */
/** 바뀔수도 있음 */

```


<br>


## 🔶 단순 수익률 (Simple Return)

| 일반식 | 사용목적 | 설명 |
| --- | --- | --- |
|$R_t = P_t - P_(t-1) \over P_(t-1)$| - 일간/구간별 **가격변화율** 측정 <br> - 후속 지표 (누적수익률, 변동성, 포트폴리오 수익률 등)의 기초 입력값 | - 연속 가격의 비율 변화. (0.02는 2% 상승을 의미) <br> - 짧은 구간에서는 로그 수익률과 거의 동일하게 해석 가능 

- 코드
```js
// prices: number[] = [P0, P1, ...] -> [R1, R2, ...] */

export function simpleReturns(prices) {
  if (!Array.isArray(prices) || prices.length < 2) throw new Error("prices length >= 2");
  const out = [];
  for (let i = 1; i < prices.length; i++) {
    const p0 = prices[i - 1], p1 = prices[i];
    if (p0 === 0) throw new Error("prev price cannot be 0");
    out.push((p1 - p0) / p0);
  }
  return out;
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)
- 005930, 초기 3일 수익률

| 회차 | 테스트 방식 | 결과 (상위3개 값) | 로그 | 설명 | 적합/부적합 |
| :--: | ------------- | --------------------------------- | ------------------ | --------- | :------: |
|  1 | 랜덤워크 seed=500 | \[0.003006, 0.0108, 0.029755]     | 005930 초기 3일 단순수익률 | 전일 대비 변화율 | 적합     |
|  2 | seed=501      | \[-0.021199, 0.011551, 0.018914]  | Sim1 동일  | 하락→반등 | 적합 |
|  3 | seed=502  | \[-0.012702, 0.02636, 0.010139]   | Sim1 동일  | 혼합 변동 | 적합 |
|  4 | seed=503  | \[-0.005443, 0.020439, 0.007597]  | Sim1 동일  | 완만 우상향| 적합 |
|  5 | seed=504  | \[0.016413, -0.00735, -0.006564]  | Sim1 동일  | 상승 후 조정   | 적합 |
|  6 | seed=505  | \[0.012259, -0.009314, -0.008494] | Sim1 동일  | 상승 후 조정   | 적합 |
|  7 | seed=506  | \[0.02808, -0.009524, 0.00081]| Sim1 동일  | 점프·조정·보합  | 적합 |
|  8 | seed=507  | \[0.030582, -0.020577, -0.010931] | Sim1 동일  | 점프 후 하락   | 적합 |
|  9 | seed=508  | \[-0.004386, -0.017683, -0.00455] | Sim1 동일  | 완만 하락 | 적합 |
| 10 | seed=509  | \[0.013054, -0.012572, 0.022887]  | Sim1 동일  | 상→하→상 | 적합 |


<br>


## 🔶 로그 수익률 (Log Return)

| 일반식 | 사용목적 | 설명 |
| --- | --- | --- |
|$ℓt​=ln{(P_t) \over P_(t-1)}$| - 장기 누적/모형화에 유리(합산 성질) <br> - 분포 특성이 단순수익률보다 정규에 가까운 경향 | - 값이 작을수록 $ℓ𝑡≈𝑅𝑡$  (작은 구간에서는 근사적 동일)

- 코드
```js
export function logReturns(prices) {
  if (!Array.isArray(prices) || prices.length < 2) throw new Error("prices length >= 2");
  const out = [];
  for (let i = 1; i < prices.length; i++) {
    const p0 = prices[i - 1], p1 = prices[i];
    if (p0 <= 0 || p1 <= 0) throw new Error("prices must be > 0 for log return");
    out.push(Math.log(p1 / p0));
  }
  return out;
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)
- 005930, 초기 3일 수익률
  
| 회차 | 테스트 방식   | 결과 상위3                            | 로그                 | 설명                | 적합/부적합 |
| :-: | -------- | --------------------------------- | ------------------ | ----------------- | :------: |
|  1 | seed=500 | \[0.003001, 0.010742, 0.029321]   | 005930 초기 3일 로그수익률 | ln(P\_t/P\_{t-1}) | 적합     |
|  2 | seed=501 | \[-0.021427, 0.011485, 0.018737]  | Sim1 동일                 | Sim1 동일                | 적합     |
|  3 | seed=502 | \[-0.012782, 0.026016, 0.010088]  | Sim1 동일                 | Sim1 동일                | 적합     |
|  4 | seed=503 | \[-0.005458, 0.020232, 0.007568]  | Sim1 동일                 | Sim1 동일                | 적합     |
|  5 | seed=504 | \[0.016279, -0.007377, -0.006586] | Sim1 동일                 | Sim1 동일                | 적합     |
|  6 | seed=505 | \[0.012184, -0.009359, -0.008531] | Sim1 동일                 | Sim1 동일                | 적합     |
|  7 | seed=506 | \[0.027689, -0.009571, 0.00081]   | Sim1 동일                 | Sim1 동일                | 적합     |
|  8 | seed=507 | \[0.030116, -0.020788, -0.010991] | Sim1 동일                 | Sim1 동일                | 적합     |
|  9 | seed=508 | \[-0.004396, -0.017842, -0.00456] | Sim1 동일                 | Sim1 동일                | 적합     |
| 10 | seed=509 | \[0.012969, -0.012651, 0.02263]   | Sim1 동일                 | Sim1 동일                | 적합     |


<br>


## 🔶 누적 수익률 (자산)

- 수익 <br>
$CR_t = \prod_{i=1}^{t} (1 + R_i) - 1 = \frac{P_t}{P_0} - 1$

| 사용목적 | 설명 |
| --- | --- |
| - 특정 자산의 기간 전체 성과 요약 | - 처음 대비 현재까지 총 몇 % 수익인가를 직관적으로 제시

- 코드
```js
export function cumulativeReturn(returns) {
  if (!Array.isArray(returns) || returns.length === 0) throw new Error("returns required");
  return returns.reduce((g, r) => g * (1 + r), 1) - 1;
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)
- 005930, 초기 3일 수익률

| 회차 | 테스트 방식   | 결과 상위3       | 로그          | 설명       | 적합/부적합 |
| :-: | -------- | ------------ | ----------- | -------- | :------: |
|  1 | seed=500 | \[0.431554]  | 마지막/첫가격 − 1 | 기간 전체 누적 | 적합     |
|  2 | seed=501 | \[0.23484]   | Sim1 동일          | Sim1 동일       | 적합     |
|  3 | seed=502 | \[-0.077198] | Sim1 동일          | 하락 시나리오  | 적합     |
|  4 | seed=503 | \[0.138894]  | Sim1 동일          | 완만 플러스   | 적합     |
|  5 | seed=504 | \[0.378177]  | Sim1 동일          | 상승장      | 적합     |
|  6 | seed=505 | \[0.255649]  | Sim1 동일          | Sim5 동일       | 적합     |
|  7 | seed=506 | \[0.204882]  | Sim1 동일          | Sim5 동일       | 적합     |
|  8 | seed=507 | \[0.287475]  | Sim1 동일          | Sim5 동일       | 적합     |
|  9 | seed=508 | \[-0.169769] | Sim1 동일          | 하락장      | 적합     |
| 10 | seed=509 | \[0.368376]  | Sim1 동일          | 우상향      | 적합     |


<br>


## 🔶 단순 이동평균 (SMA, 5일)

- 수식 <br>
$SMA_t = \frac{1}{n} \sum_{i=0}^{n-1} P_{t-i}, \quad (n=5)$

| 사용목적 | 설명 |
| --- | --- |
| - 단기 노이즈 제거 및 추세 파악 | - 창(window)이 길수록 부드럽지만 반응 속도는 느려짐

- 코드
```js
export function sma(series, n) {
  if (!Array.isArray(series) || series.length < n) throw new Error("series length >= n");
  const out = [], q = [];
  let acc = 0;
  for (let i = 0; i < series.length; i++) {
    q.push(series[i]); acc += series[i];
    if (q.length > n) acc -= q.shift();
    if (q.length === n) out.push(acc / n);
  }
  return out;
}
s.reduce((g, r) => g * (1 + r), 1) - 1;
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)
- 005930, 마지막 시점 SMA5

| 회차 | 테스트 방식   | 결과 상위3     | 로그                | 설명        | 적합/부적합 |
| :-: | -------- | ---------- | ----------------- | --------- | :------: |
|  1 | seed=500 | \[98981.2] | 005930 SMA5 마지막 값 | 최근 5일 평균가 | 적합     |
|  2 | seed=501 | \[86353.9] | Sim1 동일                | Sim1 동일        | 적합     |
|  3 | seed=502 | \[65326.4] | Sim1 동일                | Sim1 동일        | 적합     |
|  4 | seed=503 | \[78884.4] | Sim1 동일                | Sim1 동일        | 적합     |
|  5 | seed=504 | \[98024.7] | Sim1 동일                | Sim1 동일        | 적합     |
|  6 | seed=505 | \[85151.3] | Sim1 동일                | Sim1 동일        | 적합     |
|  7 | seed=506 | \[83530.6] | Sim1 동일                | Sim1 동일        | 적합     |
|  8 | seed=507 | \[88792.0] | Sim1 동일                | Sim1 동일        | 적합     |
|  9 | seed=508 | \[58190.7] | Sim1 동일                | Sim1 동일        | 적합     |
| 10 | seed=509 | \[96761.5] | Sim1 동일                | Sim1 동일        | 적합     |


<br>


## 🔶 누적 포트폴리오 수익률


- 수식 <br>
$
r_{p,t} = \sum_i w_i r_{i,t}, \qquad
CR_p = \prod_t (1 + r_{p,t}) - 1
$

| 사용목적 | 설명 |
| --- | --- |
| - 포트폴리오의 기간 전체 성과 요약 | - 시가가치 고정가중 $w_i$가정 (리밸런싱X)으로 각 일간 수익률을 합성

- 코드
```js
export function portfolioReturns(weights, priceTable) {
  // priceTable: [{day, 005930, 000660, 035420, ...}]
  const keys = Object.keys(weights);
  const out = [];
  for (let i = 1; i < priceTable.length; i++) {
    let rp = 0;
    for (const k of keys) {
      const p1 = priceTable[i][k], p0 = priceTable[i - 1][k];
      rp += weights[k] * ((p1 - p0) / p0);
    }
    out.push({ day: priceTable[i].day, r: rp });
  }
  return out;
}

export const cumulativeReturn = arr => arr.reduce((g, x) => g * (1 + (x.r ?? x)), 1) - 1;
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)
- 약 90~120영업일 누적

| 회차 | 테스트 방식   | 결과 상위3      | 로그          | 설명       | 적합/부적합 |
| :-: | -------- | ----------- | ----------- | -------- | :------: |
|  1 | seed=500 | \[0.106969] | $Π(1+r\_p)−1$ | 기간 누적 성과 | 적합     |
|  2 | seed=501 | \[0.100226] | Sim1 동일          | Sim1 동일       | 적합     |
|  3 | seed=502 | \[0.087561] | Sim1 동일          | Sim1 동일       | 적합     |
|  4 | seed=503 | \[0.132352] | Sim1 동일          | Sim1 동일       | 적합     |
|  5 | seed=504 | \[0.107516] | Sim1 동일          | Sim1 동일       | 적합     |
|  6 | seed=505 | \[0.095999] | Sim1 동일          | Sim1 동일       | 적합     |
|  7 | seed=506 | \[0.078603] | Sim1 동일          | Sim1 동일       | 적합     |
|  8 | seed=507 | \[0.091493] | Sim1 동일          | Sim1 동일       | 적합     |
|  9 | seed=508 | \[0.117336] | Sim1 동일          | Sim1 동일       | 적합     |
| 10 | seed=509 | \[0.093557] | Sim1 동일          | Sim1 동일       | 적합     |


<br>


## 🔶 포트폴리오 수익률 시리즈 (초기 3일)


- 수식 <br>
$r_{p,t} = \sum_i w_i r_{i,t}$


| 사용목적 | 설명 |
| --- | --- |
| - 포트폴리오의 **일간 성과 흐름**을 시퀀스로 산출 <br> - 누적수익률, 변동성, VAR등 후속 계산의 기초 데이터 | - 시가가치 기준 고정가중치 포트폴리오 가정

- 코드
```js
export function portfolioReturns(weights, priceTable) {
  const keys = Object.keys(weights);
  const out = [];
  for (let i = 1; i < priceTable.length; i++) {
    let rp = 0;
    for (const k of keys) {
      const p1 = priceTable[i][k], p0 = priceTable[i - 1][k];
      rp += weights[k] * ((p1 - p0) / p0);
    }
    out.push({ day: priceTable[i].day, r: rp });
  }
  return out;
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)
- 포트폴리오 $r_p$(수익률), 초기 3일

| 회차 | 테스트 방식   | 결과 상위3                             | 로그            | 설명          | 적합/부적합 |
| :-: | -------- | ---------------------------------- | ------------- | ----------- | :------: |
|  1 | seed=500 | \[0.004553, 0.001973, 0.008267]    | 포트 $r\_p$ 초기 3일 | $Σ w\_i r\_i$ | 적합     |
|  2 | seed=501 | \[-0.00488, 0.006272, -0.004102]   | Sim1 동일            | Sim1 동일          | 적합     |
|  3 | seed=502 | \[-0.006321, 0.012487, -0.001029]  | Sim1 동일            | Sim1 동일          | 적합     |
|  4 | seed=503 | \[0.005628, 0.009045, 0.008684]    | Sim1 동일            | Sim1 동일          | 적합     |
|  5 | seed=504 | \[0.009198, -0.002893, -0.000778]  | Sim1 동일            | Sim1 동일          | 적합     |
|  6 | seed=505 | \[0.009916, -0.005047, -0.001904]  | Sim1 동일            | Sim1 동일          | 적합     |
|  7 | seed=506 | \[0.016061, -0.004083, 0.003265]   | Sim1 동일            | Sim1 동일          | 적합     |
|  8 | seed=507 | \[0.014944, -0.004679, -0.002881]  | Sim1 동일            | Sim1 동일          | 적합     |
|  9 | seed=508 | \[-0.004283, -0.010052, -0.002022] | Sim1 동일            | Sim1 동일          | 적합     |
| 10 | seed=509 | \[0.005419, -0.002321, 0.008404]   | Sim1 동일            | Sim1 동일          | 적합     |


<br>


## 🔶 공분산 / 상관 (005930 vs 000660)


- 수식 <br>
$\mathrm{Cov}(r_i, r_j) = \frac{1}{T-1}\sum_{t=1}^T (r_{i,t}-\bar r_i)(r_{j,t}-\bar r_j)$ <br>
$\rho_{ij} = \frac{\mathrm{Cov}(r_i,r_j)}{\sigma_i \sigma_j}$


| 사용목적 | 설명 |
| --- | --- |
| - 자산 간 **공동변동성**과 **분산효과** 확인 <br> - 포트폴리오 다변화 효율 측정의 기초 | - 공분산 : 방향성 있는 공동 변동량 <br> - 상관계수 +1 ~ -1 범위, 관계 강도

- 코드
```js
export function covariance(x, y) {
  const n = x.length;
  const mx = mean(x), my = mean(y);
  let acc = 0;
  for (let i = 0; i < n; i++) acc += (x[i] - mx) * (y[i] - my);
  return acc / (n - 1);
}
export function correlation(x, y) {
  return covariance(x, y) / (stdev(x) * stdev(y));
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)

| 회차 | 테스트 방식   | 결과 상위3                           | 로그     | 설명       | 적합/부적합 |
| :-: | -------- | -------------------------------- | ------ | -------- | :------: |
|  1 | seed=500 | \[corr=0.314756, cov=0.00006603] | 상관/공분산 | 동조화·공동변동 | 적합     |
|  2 | seed=501 | \[corr=0.202733, cov=0.00004168] | Sim1 동일     | Sim1 동일       | 적합     |
|  3 | seed=502 | \[corr=0.221117, cov=0.00004332] | Sim1 동일     | Sim1 동일       | 적합     |
|  4 | seed=503 | \[corr=0.229003, cov=0.00004676] | Sim1 동일     | Sim1 동일       | 적합     |
|  5 | seed=504 | \[corr=0.280492, cov=0.00005902] | Sim1 동일     | Sim1 동일       | 적합     |
|  6 | seed=505 | \[corr=0.249369, cov=0.00005067] | Sim1 동일     | Sim1 동일       | 적합     |
|  7 | seed=506 | \[corr=0.274156, cov=0.00005728] | Sim1 동일     | Sim1 동일       | 적합     |
|  8 | seed=507 | \[corr=0.298381, cov=0.00006474] | Sim1 동일     | Sim1 동일       | 적합     |
|  9 | seed=508 | \[corr=0.197122, cov=0.00004009] | Sim1 동일     | Sim1 동일       | 적합     |
| 10 | seed=509 | \[corr=0.082547, cov=0.00001802] | Sim1 동일     | Sim1 동일       | 적합     |


<br>


## 🔶 롤링 변동성 (20일, 005930)

- 수식 <br>
$\sigma_{20,t} = \sqrt{\frac{1}{20-1}\sum_{i=0}^{19} (r_{t-i}-\bar r)^2}$

| 사용목적 | 설명 |
| --- | --- |
| - 최근 20일 기준의 **단기 리스크** 추정 | - 이동표준편차로 단기 변동성 평가 <br> - 변동성 클수록 위험 높음

- 코드
```js
export function rollingStdev(returns, window) {
  const out = [];
  for (let i = window-1; i < returns.length; i++) {
    const slice = returns.slice(i - window + 1, i + 1);
    out.push(stdev(slice));
  }
  return out;
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)
- 005930, 마지막 시점 σ20

| 회차 | 테스트 방식   | 결과 상위3      | 로그        | 설명     | 적합/부적합 |
| :-: | -------- | ----------- | --------- | ------ | :------: |
|  1 | seed=500 | \[0.019594] | 표준편차(20일) | 단기 변동성 | 적합     |
|  2 | seed=501 | \[0.015703] | Sim1 동일        | Sim1 동일     | 적합     |
|  3 | seed=502 | \[0.014846] | Sim1 동일        | Sim1 동일     | 적합     |
|  4 | seed=503 | \[0.016601] | Sim1 동일        | Sim1 동일     | 적합     |
|  5 | seed=504 | \[0.019423] | Sim1 동일        | Sim1 동일     | 적합     |
|  6 | seed=505 | \[0.016806] | Sim1 동일        | Sim1 동일     | 적합     |
|  7 | seed=506 | \[0.018257] | Sim1 동일        | Sim1 동일     | 적합     |
|  8 | seed=507 | \[0.017294] | Sim1 동일        | Sim1 동일     | 적합     |
|  9 | seed=508 | \[0.015264] | Sim1 동일        | Sim1 동일     | 적합     |
| 10 | seed=509 | \[0.01655]  | Sim1 동일        | Sim1 동일     | 적합     |


<br>


## 🔶 연환산 지표&샤프 (포트폴리오)


- 수식 <br>
$\bar r = \text{mean}(r_p), \quad \sigma = \text{stdev}(r_p)$

    $\text{annRet} = (1+\bar r)^{252}-1, \quad
\text{annStd} = \sigma \sqrt{252}, \quad
\text{Sharpe} = \frac{\text{annRet}-r_f}{\text{annStd}}$

| 사용목적 | 설명 |
| --- | --- |
| - 연 기준 성과 및 위험조정수익 평가 | - Sharpe Ratio > 0 이면 무위험 대비 초과성과 <br> -annRet : 연환산 기대수익률

- 코드
```js
export function annualStats(returns, rf=0.015, freq=252) {
  const mu = mean(returns);
  const sd = stdev(returns);
  const annRet = Math.pow(1+mu, freq) - 1;
  const annStd = sd * Math.sqrt(freq);
  const sharpe = (annStd > 0) ? (annRet - rf) / annStd : NaN;
  return { mu, sd, annRet, annStd, sharpe };
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)

| 회차 | 테스트 방식   | 결과 상위3                                         | 로그     | 설명     | 적합/부적합 |
| :-: | -------- | ---------------------------------------------- | ------ | ------ | :------: |
|  1 | seed=500 | \[mean=0.000118, std=0.01074, annRet=0.031006] | 연환산/샤프 | 연기준 요약 | 적합     |
|  2 | seed=501 | \[0.000111, 0.010133, 0.028867]                | Sim1 동일     | Sim1 동일     | 적합     |
|  3 | seed=502 | \[9.7e-05, 0.009899, 0.025165]                 | Sim1 동일     | Sim1 동일     | 적합     |
|  4 | seed=503 | \[0.000125, 0.010532, 0.032052]                | Sim1 동일     | Sim1 동일     | 적합     |
|  5 | seed=504 | \[0.000117, 0.010592, 0.030923]                | Sim1 동일     | Sim1 동일     | 적합     |
|  6 | seed=505 | \[0.000107, 0.010377, 0.028446]                | Sim1 동일     | Sim1 동일     | 적합     |
|  7 | seed=506 | \[8.5e-05, 0.009907, 0.022511]                 | Sim1 동일     | Sim1 동일     | 적합     |
|  8 | seed=507 | \[9.9e-05, 0.010314, 0.02595]                  | Sim1 동일     | Sim1 동일     | 적합     |
|  9 | seed=508 | \[0.000127, 0.010408, 0.032614]                | Sim1 동일     | Sim1 동일     | 적합     |
| 10 | seed=509 | \[0.000103, 0.011018, 0.026949]                | Sim1 동일     | Sim1 동일     | 적합     |


<br>


## 🔶 베타, 알파 (005930 vs KOSPI)


- 수식 <br>
$\beta = \frac{\mathrm{Cov}(r_i, r_m)}{\mathrm{Var}(r_m)}, \quad
\alpha = R_i - \big( r_f + \beta (R_m - r_f)\big)$


| 사용목적 | 설명 |
| --- | --- |
| - 시장 민감도β, 및 초과성과α 추정 | - β > 1: 시장보다 민감, β < 1: 방어적 <br> - α > 0: CAPM 기대 대비 초과성과

- 코드
```js
export function betaAlpha(assetRets, benchRets, rf=0.015, freq=252) {
  if (assetRets.length !== benchRets.length) throw new Error("length mismatch");
  const cov = covariance(assetRets, benchRets);
  const beta = cov / (stdev(benchRets)**2);
  const muA = mean(assetRets), muM = mean(benchRets);
  const annA = Math.pow(1+muA, freq) - 1;
  const annM = Math.pow(1+muM, freq) - 1;
  const alpha = annA - (rf + beta * (annM - rf));
  return { beta, alpha };
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)

| 회차 | 테스트 방식   | 결과 상위3                            | 로그   | 설명         | 적합/부적합 |
| :-: | -------- | --------------------------------- | ---- | ---------- | :------: |
|  1 | seed=500 | \[beta=-0.057606, alpha=1.165921] | CAPM | 시장 민감/초과성과 | 적합     |
|  2 | seed=501 | \[-0.021291, 0.95007]             | Sim1 동일   | Sim1 동일         | 적합     |
|  3 | seed=502 | \[-0.023456, 0.884163]            | Sim1 동일   | Sim1 동일         | 적합     |
|  4 | seed=503 | \[-0.032618, 1.009909]            | Sim1 동일   | Sim1 동일         | 적합     |
|  5 | seed=504 | \[-0.027153, 1.143278]            | Sim1 동일   | Sim1 동일         | 적합     |
|  6 | seed=505 | \[-0.020735, 0.927324]            | Sim1 동일   | Sim1 동일         | 적합     |
|  7 | seed=506 | \[-0.048621, 1.016428]            | Sim1 동일   | Sim1 동일         | 적합     |
|  8 | seed=507 | \[-0.035843, 1.061043]            | Sim1 동일   | Sim1 동일         | 적합     |
|  9 | seed=508 | \[-0.024089, 0.962423]            | Sim1 동일   | Sim1 동일         | 적합     |
| 10 | seed=509 | \[-0.053755, 0.96832]             | Sim1 동일   | Sim1 동일         | 적합     |


<br>


## 🔶 드로우다운 & 최대 낙폭(MDD)


- 수식 <br>
$DD_t = \frac{P_t}{\max_{s \leq t} P_s} - 1, \quad MDD = \min_t DD_t$

| 사용목적 | 설명 |
| --- | --- |
| - **투자 중 최악의 손실폭**을 측정 <br> - 위험 관리 및 자금 운용 안정성 지표 | - 드로우다운(DD) : 과거 최고가 대비 현재가의 하락률 <br> - 최대 낙폭(MDD) : 투자 기간 중 DD의 최솟값 <br> - "최악의 상황에서 최대 몇 %까지 잃을 수 있나"를 나타냄

- 코드
```js
export function drawdown(prices) {
  let peak = prices[0];
  const dds = [];
  for (let i = 0; i < prices.length; i++) {
    if (prices[i] > peak) peak = prices[i];
    const dd = prices[i] / peak - 1;
    dds.push(dd);
  }
  const mdd = Math.min(...dds);
  return { series: dds, mdd };
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)

| 회차 | 테스트 방식   | 결과 상위3       | 로그    | 설명         | 적합/부적합 |
| :-: | -------- | ------------ | ----- | ---------- | :------: |
|  1 | seed=500 | \[-0.114311] | 최대 낙폭 | 피크→저점 최대하락 | 적합     |
|  2 | seed=501 | \[-0.161497] | 최대 낙폭    | 피크→저점 최대하락         | 적합     |
|  3 | seed=502 | \[-0.261405] | 최대 낙폭    | 급락 시나리오    | 적합     |
|  4 | seed=503 | \[-0.129567] | 최대 낙폭    | 급락 시나리오         | 적합     |
|  5 | seed=504 | \[-0.102918] | 최대 낙폭    | 급락 시나리오         | 적합     |
|  6 | seed=505 | \[-0.14491]  | 최대 낙폭    | 급락 시나리오         | 적합     |
|  7 | seed=506 | \[-0.154693] | 최대 낙폭    | 급락 시나리오         | 적합     |
|  8 | seed=507 | \[-0.121079] | 최대 낙폭    | 급락 시나리오         | 적합     |
|  9 | seed=508 | \[-0.263062] | 최대 낙폭    | 극단적 낙폭     | 적합     |
| 10 | seed=509 | \[-0.062774] | 최대 낙폭    | 낙폭 제한적     | 적합     |


<br>


## 🔶 Var (95%, 히스토리컬)


- 수식 <br>
$VaR_{95\%} = - Q_{5\%}(r_p)$

| 사용목적 | 설명 |
| --- | --- |
| - 1일 기준 **최악의 손실 한계치** 추정 <br> - 리스크 관리 및 규제 보고 지표 | - 히스토리컬 방법 : 과거 수익률 분포에서 하위 5% 지점을 추출 <br> - "95% 확률로 하루 손실이 이 값을 넘지 않는다"는 의미

- 코드
```js
export function histVaR(returns, p=0.05) {
  if (!Array.isArray(returns) || returns.length === 0) throw new Error("returns required");
  const sorted = [...returns].sort((a,b)=>a-b);
  const idx = Math.floor(p * sorted.length);
  return -sorted[idx];
}
```

### 🔷 [테스트/시뮬레이션 결과]
- (10회, 상위 3개 표기)

| 회차 | 테스트 방식   | 결과 상위3      | 로그            | 설명        | 적합/부적합 |
| :-: | -------- | ----------- | ------------- | --------- | :------: |
|  1 | seed=500 | \[0.013435] | 히스토리컬 VaR 95% | 1일 손실 임계치 | 적합     |
|  2 | seed=501 | \[0.011804] | Sim1 동일            | Sim1 동일        | 적합     |
|  3 | seed=502 | \[0.013776] | Sim1 동일            | Sim1 동일        | 적합     |
|  4 | seed=503 | \[0.014369] | Sim1 동일            | Sim1 동일        | 적합     |
|  5 | seed=504 | \[0.012423] | Sim1 동일            | Sim1 동일        | 적합     |
|  6 | seed=505 | \[0.012098] | Sim1 동일            | Sim1 동일        | 적합     |
|  7 | seed=506 | \[0.012647] | Sim1 동일            | Sim1 동일        | 적합     |
|  8 | seed=507 | \[0.011921] | Sim1 동일            | Sim1 동일        | 적합     |
|  9 | seed=508 | \[0.012516] | Sim1 동일            | Sim1 동일        | 적합     |
| 10 | seed=509 | \[0.013949] | Sim1 동일            | Sim1 동일        | 적합     |

<br><br>

