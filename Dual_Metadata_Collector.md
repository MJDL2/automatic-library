# 📚 Dual Metadata Collector

**목표:**  
ISBN을 기반으로 국립중앙도서관(LNK)의 Linked Open Data (LOD) 시스템에서 도서 메타데이터를 자동 수집하여 JSON으로 저장하는 파이썬 스크립트.

---

## ✅ 작동 원리 요약

1. 사용자가 ISBN을 입력
2. LOD API 호출 → `uri` (책 리소스 ID)만 추출
3. `https://lod.nl.go.kr/page/{KMO코드}` HTML 페이지 접근
4. RDF 속성 기반으로 `<table class="table2">` 내에서 주요 메타데이터 필드 파싱
5. JSON으로 저장 (파일명: `{ISBN}_metadata.json`)

---

## 🔍 수집 대상 메타데이터 필드

| RDF URI 포함 키워드       | 저장 필드 (`metadata`)     |
|---------------------------|-----------------------------|
| `dcterms:title`           | `title`                    |
| `dc:creator`              | `author`                   |
| `titleOfSeries`           | `series`                   |
| `dc:publisher`            | `publisher`                |
| `nlon:datePublished`      | `published_date`           |
| `nlon:kdc`                | `kdc`                      |
| `nlon:kdcn`               | `kdcn`                     |
| `bf:extent`               | `extent`                   |
| `nlon:publicationPlace`   | `publication_place`        |
| `dcterms:description`     | `description`              |

---

## 🛠 주요 기술 스택

- Python 3
- `requests` / `BeautifulSoup` (HTML 파싱)
- `isbnlib` (ISBN 유효성 검사)
- Markdown + JSON 출력

---

## 📁 결과 예시

```json
{
  "isbn": "9788936434120",
  "metadata": {
    "title": "소년이 온다",
    "author": "한강",
    "publisher": "창비",
    "published_date": "2014",
    "kdc": "813.62",
    ...
  }
}
```

---

## 🧠 기타 메모

- LOD API는 단순히 `uri` 추출용으로만 활용
- 실제 서지 데이터는 HTML 페이지(`https://lod.nl.go.kr/page/...`)에서 수집
- HTML 구조 변경 시 파싱 규칙도 조정 필요

---

🗓 문서 생성일: 2025-03-26

---





- 전체 구조를 2개의 모듈로 분리:
  - `isbn_to_nlk_uri.py`: ISBN을 기반으로 NLK API에서 도서 URI 추출 및 페이지 URL 변환
  - `metadata_uri.py`: 위 URL로 HTML에 접근하여 상세 메타데이터 파싱 및 추출
- KDC 분류번호가 정확히 추출되도록 `prop_uri.endswith('/kdc')` 방식 도입
- 수집된 메타데이터를 `metadata/` 폴더에 자동 저장
- 저장된 JSON 파일을 OS에 맞게 자동으로 열도록 구현
  - Windows: `os.startfile()`
  - macOS: `open`
  - Linux: `xdg-open`
- 사용자 경험 개선:
  - 실행 중 절대경로 출력
  - 저장 완료 메시지 출력


---

---

## 🛠 2025-03-26 업데이트 정리

### 📌 주요 개선 및 기능 확장

1. **모듈 구조 개선**
   - `isbn_to_nlk_uri.py`: ISBN으로 NLK URI 추출
   - `metadata_uri.py`: URI에 대응하는 HTML 페이지에서 메타데이터 파싱 및 저장

2. **LOD 페이지 구조 대응 강화**
   - `<ul>` 안에 `<li>`가 여러 개인 구조에서 `extent` 항목 (`21 cm`, `215 p`)을 모두 인식
   - `형태사항: "215쪽, 21cm"` 형태로 저장 가능

3. **불필요한 문자열 정제**
   - `(xsd:string)`, `"` 따옴표 등 제거

4. **MARC 기반 필드 정렬 및 용어 매핑**
   - `title` → `서명`, `author` → `저자명` 등 도서관용 명칭으로 치환
   - 저장 순서를 MARC 흐름에 맞춰 정리:
     - 서명 → 저자명 → 출간지 → 출판사 → 출판일 → 형태사항 → KDC 분류번호

5. **자동 열기 기능 제거**
   - JSON 저장은 유지하되 파일 자동 실행은 생략하여 CLI 중심으로 전환

6. **label 필드로부터 서명/저자 자동 분리**
   - `소년이 온다 / 지은이: 한강` → `서명: 소년이 온다`, `저자명: 한강`

---

🧷 향후 과제
- ISBN 일괄 처리 기능
- MARC21 포맷 또는 USMARC 변환기 연동
- Streamlit/CLI 기반 시각 인터페이스 확장
