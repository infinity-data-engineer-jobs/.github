![de_md](https://github.com/user-attachments/assets/2076aa33-8bb7-47ad-b1a2-5a6a8ca91a84)

# 1. 프로젝트 개요

본 프로젝트는 데이터 엔지니어링 부트캠프의 1차 프로젝트로, "원티드" 채용 플랫폼에서 데이터를 수집하여 데이터 엔지니어링 분야에서 요구되는 역량과 최신 트렌드를 시각화하는 웹 애플리케이션을 개발하는 것을 목표로 합니다. 이를 통해 데이터 엔지니어링 관련 직무에서 요구되는 기술 스택, 경험, 그리고 트렌드를 분석하고, 이를 직관적으로 시각화하여 사용자들이 현업에서 필요로 하는 기술 역량을 한눈에 파악할 수 있도록 지원합니다.  

<br>

# 2. 프로젝트 주제 선정 이유

데이터 엔지니어링을 배우는 과정에서 가장 중요한 첫 번째 단계는 **현업에서 실제로 어떤 일을 하고, 어떤 기술 스택을 사용하며, 어떤 역량을 요구하는지** 정확히 파악하는 것입니다. 이를 바탕으로 학습 방향을 설정하고, 필요한 기술을 효과적으로 습득할 수 있기 때문입니다. 그러나 데이터 엔지니어링 분야는 매우 넓고 다양한 기술들이 존재하여, 초기에는 무엇부터 배워야 할지 막막할 수 있습니다.
따라서 본 프로젝트는 "원티드" 채용 플랫폼에서 데이터 엔지니어링 관련 공고를 수집하여, 현업에서 요구하는 기술 스택, 주요 업무, 우대 조건 등을 **시각적으로 분석하고 한눈에 파악할 수 있도록 하는 것**을 목표로 하였습니다. 이를 통해 데이터 엔지니어링 분야의 트렌드를 이해하고, 우리가 배워야 할 핵심 기술을 명확하게 정리함으로써, 효율적인 학습 경로를 제시하고자 하였습니다.

<br>

# 3. 라이브러리 및 환경
- **Data**
    - `Selenium`, `BeautifulSoup`, `GPT API`
- **Backend**
    - `Django`, `SQLite`
- **Frontend**
    - `React`, `tailwind css`
- **Collaboration tool**
    - `Git`, `Slack`, `Notion`, `ZEP`

<br>

# 4. 개발

## 데이터 준비 (1) : 크롤링

### 라이브러리

- selenium, beautifulsoup

### 작동 원리

- Selenium으로 페이지를 로드 후 스크롤
- 각 공고별 “상세 정보” 팝업 열기 → BeautifulSoup으로 파싱( 🌟)
- 공고의 각 항목(주요업무, 자격요건, 우대사항 등) 추출
- 회사 페이지로 이동 → 회사정보(연봉·인원수·매출 등) 추출
- SQLite DB에 테이블 생성 → 삽입


## 데이터 준비 (2) : 전처리

### 기술 스택

<aside>

📜기술 스택 : charts_noticeInfo.notice_tech_stack

처리 방식 : 

- 크롤링한 데이터의 “기술 스택” 항목 내 정의된 데이터들을 notice_tech_stack필드로 Insert
- 데이터 엔지니어가 사용할만한 기술스택을 미리 정의 후 “주요 업무”, “우대 사항”, “포지션 상세” 항목의 텍스트 데이터에서 해당 기술스택 항목을 추출하여 notice_tech_stack필드로 Insert
</aside>

<br>

### 우대 사항

<aside>

📜우대 사항 : charts_noticeinfo.preferred_qualificaton

처리 방식

- 크롤링 데이터의 “우대 사항” 또는 “우대 조건”에 대한 데이터를 charts_noticeinfo의 preferred_qualification 필드에 저장
- charts_noticeinfo의 preferred_qualificaton 컬럼의 데이터를 문장 단위로 슬라이싱
- 슬라이싱된 문장 데이터를 GPT API 를 통해서 유사한 의미의 문장으로 그룹화 진행
- representative_sentence, frequency, sentences 로 나뉘어진 json 포맷의 결과 데이터를 charts_preferred_qualification_info 테이블을 생성하고 저장
</aside>

<br>

### 주요 업무 

<aside>

📜주요 업무 : charts_noticeinfo.notice_main_work

처리방식

- 데이터 전처리 과정에서 필터링된 채용 공고들을 대상으로, 각 공고의 **‘주요 업무(main_work)’ 항목을 행 단위로 분리하여** 처리
- 이후 각 업무 텍스트를 GPT 모델을 통해 자동 분류하고, 그 결과를 별도의 테이블에 저장하는 과정을 수행

GPT API 
- `classify_and_insert()` 함수는 주요 업무 텍스트를 GPT 모델에 보내어 **사전 정의된 6개의 업무 유형 중 하나로 분류**합니다.
- 프롬프트에는 다음과 같은 6개 업무 유형이 정의되어 있으며, 해당 텍스트가 가장 가까운 항목의 번호를 하나만 출력하도록 요청합니다:
    1. ETL/ELT 설계 및 자동화
    2. 분산 처리 시스템 구축
    3. 클라우드 인프라 및 컨테이너 운영
    4. BI 시스템 및 데이터 분석 환경 구축
    5. 백엔드 개발 및 데이터 연계
    6. 딥러닝/머신러닝 역량 적용
        
        (해당 없음일 경우 0번)
        
- 분류 요청은 `GPT-4.1-nano` 모델을 사용하며, `temperature=0`으로 설정하여 **일관된 결과**를 유도하였습니다.

<br>

### 회사 규모

<aside>

📜회사 규모 : charts_companyinfo.company_headcount

처리방식

- 데이터 전처리 과정에서 필터링된 채용 공고의 회사 정보를 대상으로 크롤링 하여 회사 정보(charts_companyinfo)테이블로 데이터 추출하여 삽입
- 항목 내 정의된 데이터 중 회사 인원 텍스트 데이터를 charts_companyinfo.company_headcount로 Insert
- 대시보드에 필요한 회사 인원 데이터를 그루핑하여 일정 규모로 데이터 추출
</aside>

<br>

## 백엔드 구성

🎨구현 목적 : Django ORM을 활용한 데이터 모델 구성, API 호출을 통한 크롤링 작업 실행 구현, REST API를 통한 서버-클라이언트간 데이터 전달 과정을 구성하였습니다.

🚀Database ERD
![Image](https://github.com/user-attachments/assets/55a9d325-599d-4921-9d00-0e38d501d10e)


## 프론트엔드 구성
🎨구현 목적 : 데이터 엔지니어와 관련된 채용 공고를 수집한 뒤, 그 내용을 시각화하여 사용자가 직관적으로 직무 정보를 파악할 수 있도록 하기 위함입니다. 사용자에게 데이터를 설명하는 데 있어 텍스트 이상의 전달력을 확보하기 위해 막대 차트 및 워드 클라우드 형태의 시각화를 도입했습니다.

✅ 전체 구조 및 구성 요소
  1. **주요 업무**
      - 데이터 엔지니어들이 실제 공고에서 수행하는 업무들을 바 차트로 시각화
      - API: `/mainWorkChartData/`
  2. **우대 사항**
      - 채용 공고에서 자주 등장하는 우대 조건을 빈도순으로 정렬 후 시각화
      - API: `/preferredQualificationData/`
  3. **회사 규모**
      - 데이터 엔지니어를 채용하는 회사들의 규모 분포를 시각화
      - API: `/headcountDistributionData/`
  4. **기술 스택**
      - 상위 10개의 기술 스택을 바 차트와 워드 클라우드 두 가지 방식으로 표현
      - API: `/tech_Stack_data/`
    
✅ 각 컴포넌트 특징

  - `Bar` 컴포넌트 (`react-chartjs-2`)를 활용한 **수평/수직 막대 차트**
      - 각 데이터셋은 백엔드 API에서 불러온 데이터를 기반으로 동적으로 생성
      - 라벨(축 정보) 및 값, 그라디언트 배경 등 시각 효과 반영
  - `WordCloud` 컴포넌트 (`react-d3-cloud`)를 활용한 **기술 스택 워드 클라우드**
      - 기술의 등장 빈도에 따라 크기 및 회전 각도를 조절하여 가시성 강화
  - 각 카드 영역은 TailwindCSS 기반의 유연한 그리드 레이아웃과 디자인을 통해 **반응형 구성**으로 구현됨

🔁 데이터 연동 방식

  - 모든 데이터는 **React의 `useEffect` 훅 내에서 fetch API**를 통해 백엔드에서 비동기적으로 호출
  - 데이터 로딩 시 `loading` 상태를 두어 **UX 개선**
  - 로딩이 끝난 후에는 `useState` 훅으로 상태를 관리하여 컴포넌트 재렌더링을 유도


# 5. 결과
## 시각화 세부 결과
<img width="1343" alt="Image" src="https://github.com/user-attachments/assets/ce7bf0b7-f83b-42bb-b397-dd6bb5587a57" />
