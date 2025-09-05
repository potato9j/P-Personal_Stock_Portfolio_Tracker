# 국내 주식 포트폴리오 분석 웹앱 (Stock Portfolio Analyzer)

## 프로젝트 개요
- 목표 :  **국내 주식 포트폴리오 분석 웹 애플리케이션**을 개발
- 구체 목표 : 국내 주식 데이터(API)를 가져와 포트폴리오 수익률/종묵/비중/리스크 지표를 계산/시각화하여 투자 의사결정을 돕는 간단한 실용형 도구 제작. 
 - 사전 필요 지식 : 금융 공학 (투자설계, 채권분석), 기초적인 회계 IS/BS지식

- 구동 플랫폼 : GitHub Pages (`github.io`)를 사용하여 무료로 배포하며, 링크를 통해 쉽게 공유
---

<br>

## 주요 기능
- **국내 주식 실시간 데이터 수집 (API)**  [*참고: 크롤링은 사이트 약관 준수 필요*]
  - [네이버 금융](https://finance.naver.com/) 
  - [KIS Developers (한국투자증권)](https://apiportal.koreainvestment.com/) 
  - [KIS Developers Github (한국투자증권 깃허브)](https://github.com/koreainvestment/open-trading-api)  
- **포트폴리오 입력**
  - 종목 코드 / 매입가 / 수량 입력, CRUD
- **포트폴리오 분석(평가지표)**
  - 현재 평가금액, 수익률, 종목별 비중 계산
- **차트 시각화**
  - Pie Chart: 종목별 비중  
  - Line Chart: 포트폴리오 수익률 추이
- **반응형 UI**
  - 모바일·PC 대응
- **다운로드/공유**
  - 간단한 CSV 다운로드 기능  
  - `github.io` 링크로 공유 가능

- **다크모드 및 모바일 최적화** [*옵션*]
  - 반응형 UI제공

---

<br>

## 기술 스택
| 구분 | 기술 |
|-------|-------|
| **Frontend** | React.js, Tailwind CSS, Recharts(차트) |
| **Backend(API처리)** | Node.js (단순 프록시 서버) |
| **Serverless Proxy** | Vercel Serverless Functions (KIS API Key 보호용) |
| **Data** | KIS Developers API (권장) / *사전가공JSON (대안)* |
| **배포** | GitHub Pages / GitHub Actions |
| **버전 관리** | Git |
| **추가 라이브러리** | Axios (API 요청), localStorage (클라이언트 저장), Framer Motion (애니메이션), File export (CSV) |
> 불필요 및 과한 의존성 제외 : Puppeteer, complexity Library etc.

---

<br>

## 개발 로드맵 (10주 플랜)

| 주차 | 목표 | 세부작업 | 필요기술 | 
|:---:|:---:|---|:--:|
| 1주차 | 환경 세팅, 기본구조 | - GitHub repository/branch 전략(main/dev) 구성 <br> - Vite + React + TypeScript Project <br> Tailwind 설정 (PostCSS/Autoprefixer) 및 default color, font, token <br> - GitHub Pages WorkFlow (gh-pages or Actions) 1차 연동 테스트 <br> - ESLint/Prettier Rull Setup | Git, Node/npm, React, Tailwind, GitHun Pages
| 2주차 | UI 스킬레톤, 상태모델 | - 페이지 뼈대 : Home/PortFolio/Report Routing 구성 (React Router) <br> - Layout Component (Nav/Footer/Container) 생성 <br> - 상태 모델 정의 : 포트폴리오(positions[]), 시세(quites), UI상태(loading/error) <br> - localStroage 저장/복원 유틸 작성(`load/savePortfolio`) <br> - Form스펙 문서화 (필수 필드, 유효성 규칙, 에러 메시지) | React Router, localStorage |
| 3주차 | 데이터 경로 확정, Proxy | - 데이터 전략 결정 : (A) KIS API via Vercel, (B) 사전가공 JSON <br> - Vercel Serverless `api/quotes` EndPoint Prototype (심볼 > 현재가/전일비ㅣ/거래량) <br> - Acios 인스턴스, 공통 에러 인터셉터, 재시도 정책 (지수 백오프) 설정 <br> - 심볼 표준화 (숫자 6자리, 공백 제거), 기본 검증 로직 | Acios, Vercel Serverlsess |
| 4주차 | 입력, 검증, CRUD | - 포지션 입력폼 (심볼/매입가/수량) - 즉시 검증 (숫자/양수) <br> - 추가/수정/삭제/리셋 CRUD완성 및 localStrage 동기화 <br> - 심볼 입력 시 자동 보정 (하이픈 제거, 0패딩) <br> - 에러 토스트/인라인 에러 UX | React(폼 제어), localStorage | 
| 5주차 | 분석식 1차 | - 평가금액/평가손익/수익률(개별,총합) 계산 유틸 제작 <br> - 종목별 비중(현재가*수량/총합) 계산 및 소수점 처리 기준 정의 <br> - 데이터 결측/호출 실패 시 대체 로직 (마지막 유효가/ 사용자 확인) | JS 계산 유틸 | 
| 6주차 | 차트 시각화 | - Rechats Pi(비중), Line(수익률 추이) 컴포넌트 구현 <br> - 반응형 컨테이너, 툴팁/범례/포맷터(%,\) 구현 <br> - 접근성 고려 : 색맹 팔레트/대체텍스트/키보드포커스 *(옵션)* | Rechats | 
| 7주차 | UX안정화 | - 로딩/에러 상태 일관 처리 (스피너, 재시도 버튼) <br> - 폼 사용성 : 최근 입력값 유지, 엔터 제출, 포커스 이동 *(옵션)* <br> 빈 상태(Empty State) 가이드 (예시 스크린/데이터) | React | 
| 8주차 | 성능, 네트워크 최적화 | - API호출 최소화(심볼 디바운스/중복 요청 합치기) <br>  - 코드 스플리팅/지연로딩 적용 <br> lighthouse 측정 후 개선 (이미지/번들 축소) | Code Splitting, Lighthouse | 
| 9주차 | 문서, 배포 자동화 | - `README` 사용 가이드/스크린샷/GIT추가  <br> gh-pages or GitHub Actions로 배포 자동화 파이프라인 정리 <br> - 버전 태깅(v1.0.0) (실배포 버전 태깅 v2.0.1시작) | GitHub Pages Actions, Markdwon | 
| 10주차 | 확장, 마감 | - CSV내보내기 (UTF-8, 헤더 정의, 소수점 자리 고정) <br> = 섹터 비중 (사전매핑 JSON) - 기본 변동성 (표준편차) 추가 <br> - 최종 리팩토링 (네이밍/주석/타입 정리) | CSV export | 


---

<br>


## 아키텍처 개요
- **정적 프론트엔드 (GitHub Pages)** ↔ **Serverless Proxy (Vercel)** ↔  **KIS API**
- API Key/Secrey 프론트 노출 금지 (CORS/LateLimit 대응)
- 교육 목적일 경우 **사전가공 JSON**을 `/public/data/*.json`으로 제공
---
<br>

## 계산 로직 (요약)
- 평가금액(개별) = 현재가 * 수량
- 평가손익(개별) = (현재가 - 매입가 ) * 수량
- 수익률(개별) = (현재가 - 매입가) / 매입가
- 비중(개별) = 평가금액(개별) / 평가금액(총합)
- 기본 변동성(개별) = 과거 수익률 표준편차 (선택 구현; 데이터 소스 필요)
> 구체적인 계산 로직은 금융 공학 활용 (Ex. CAPM모형 etc.) > *`Calculation_Logic` 파일 참고*
---
<br>

## 검증, 예외 처리 규칙 
- 심볼 형식 : 6자리 숫자 외 입력 차단 / 자동 보정(0패딩)
- 수량/가격 : 음수, 0 절대 불가, 소수점 자릿수 제한
- API 실패 : n초 후 재시도, 3회 실패 시 안내 및 마지막 유효 데이터 사용 옵션
- 로컬 저장 : 최신 스키마 버전 유지 *(버전 필드로 마이그레이션)*
---
<br>

## 보안정책
- API키 보안 (Vercel 환경변수로 관리)
- 배포 시, 이용 약관/APIlimit 준수 (증권사/포털 확인)
- 개인정보 미수집 (LocalStrage 사용 시 사용자 안내)
    - 개인정보 수집 시, `PRIVATE_POLICY.md`으로 개인정보처리방침 안내
---
<br>

## 테스트 계획 (샘플)
- 각 로직에 대한 JEST 시뮬 계획 : 모의 데이터, 10회, 부/적합 판별
- 유닛 테스트 (계산 유틸) : 수익률/비중/반올림 규칙
- 통합 테스트 (입력>분석>시각화) : 폼 > 상태 > 차트 파이프라인
- 네트워크 테스트 : 지연/타임아웃/오류코드 (4xx/5xx)
- 접근성 체크 : 키보드 내비, 대비, aria-label
---
<br>
  
## 성능 품질 체크리스트 
- LCP < 2.5s / 번들 크기 최소화
- 이미지/아이콘 SVG 사용, 코드 스플리팅 적용
- API 호출 디바운스 / 배치 처리
- 오류 / 빈 상태 UI 제공
- 모바일(360px) / 데스크탑 (>=1280px) 레이아웃 검증 
---
<br>

## 데이터 소스&한계
- KIS Developers API 사용 : 안정, 합법, 문서화 / 리밋, 사용범위는 홈페이지 참조
- 네이버 금융 크롤링 : 학습/개인 사용시 한정 권장. *(약관 위반/변경 리스크 존재)*
- 사전가공 JSON : 강의, 교육자료 적합, 실시간성, 사용 범위 제한
---
<br>

## 사용 가이드 (요약)
1. 포지션 추가(심볼/매입가/수량) > 저장
2. "데이터 갱신" 클릭 > 현재가/비중/수익률 갱신
3. 보고서 탭에서 요약 지표, 차트 확인
4. CSv내보내기로 기록 보관
---
<br>

## 활용 방안
- **개인 투자 관리**: 종목별 비중 및 수익률 분석
- **포트폴리오 평가**: 투자 포트폴리오 리밸런싱 참고
- **포트폴리오 증빙**: 포트폴리오를 CSV로 저장하여 기록 관리
- **포트폴리오 공유**: 친구나 멘토와 링크 공유 가능
- **포트폴리오 학습용 자료**: React 및 API 연동 학습에 최적화

---

<br>

## 포트폴리오에 어필할 포인트
- 국내 API 연동 경험 (KIS Developers, 네이버 금융)
- React 기반 **실시간 데이터 처리**와 **상태 관리**
- 시각화 라이브러리 활용 능력 (Recharts)
- GitHub Pages 배포 및 CI/CD 경험
- 성능 최적화 및 반응형 디자인 구현 경험

<br>

### LICENSE - 