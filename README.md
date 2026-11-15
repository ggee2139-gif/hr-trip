# 홍콩 가족여행 웹앱 (11/18–11/22)

휴대폰에서 실시간 날씨, MTR, A31 버스, 교통 소식, 근처 먹거리를 보는 개인용 웹앱입니다.
빌드 도구나 서버 코드 없이 파일 2개(`index.html`, `foods.json`)로 동작합니다.

## 파일
- `index.html` — 앱 전체 (화면, 스타일, 로직)
- `foods.json` — 먹거리 목록 (직접 수정 가능)

## 데이터 출처 (2026-11-14에 실제 호출로 확인)
모두 무료, API 키 없음, `Access-Control-Allow-Origin: *` 응답이라 브라우저에서 바로 호출합니다. 프록시가 필요 없습니다.

| 기능 | 주소 | 비고 |
|---|---|---|
| 현재 날씨 | `https://data.weather.gov.hk/weatherAPI/opendata/weather.php?dataType=rhrread&lang=en` | 홍콩천문대. 지점별 기온 중 `Tsuen Wan` 지점을 우선 표시 |
| 9일 예보 | `...weather.php?dataType=fnd&lang=en` | 오늘·내일만 표시. `PSR` = 강수 가능성 |
| 경보 요약 | `...weather.php?dataType=warnsum&lang=en` | 경보 없으면 `{}`. 경보 코드별로 조언 표시 (`TC8` 이상·흑색 호우는 일정 취소 권장) |
| MTR 다음 열차 | `https://rt.data.gov.hk/v1/transport/mtr/getSchedule.php?line=노선&sta=역&lang=EN` | 응답의 `time`(HKT) 기준으로 남은 분 계산 |
| A31 버스 도착 | `https://data.etabus.gov.hk/v1/transport/kmb/eta/정류장ID/A31/1` | **A31(Long Win)은 KMB API에 포함**. 응답에 양방향이 섞여 있어 `dir`로 거름 |
| 특별 교통 소식 | `https://resource.data.one.gov.hk/td/en/specialtrafficnews.xml` | 교통국. XML의 `EngShort`, `ReferenceDate` 사용 |

### 확인한 코드
- MTR 역: 툰마선 `TML`-`TWW`(췬완웨스트, 숙소), `TML`-`NAC`(남청), `TML`-`ETS`(이스트침사추이), 디즈니선 `DRL`-`SUN`(써니베이), `DRL`-`DIS`(디즈니랜드 리조트), 췬완선 `TWL`-`CEN`(센트럴)
- A31 정류장: 출발 `708870108A511AF7` = 如心廣場巴士總站 (Nina Tower 버스 터미널, 숙소 건물), 공항 `7CB61EFB024D9025` = 機場地面運輸中心 (GTC)
  - 공항→숙소는 inbound(`dir: I`), 숙소→공항은 outbound(`dir: O`)
  - `rmk_en`이 `Scheduled Bus`이면 실시간 위치가 아닌 시간표 기준 → 화면에 "예정"으로 표시

## 동작 방식
- 날씨 탭: 홍콩 현재·예보 + 서울 비교(wttr.in, 키 없음) + 홍콩 기온 기준 옷차림 추천
- 탭: 날씨 / 교통 / 먹거리. 경보는 어느 탭에서든 맨 위에 표시
- 자동 새로고침: 날씨·경보 10분, 교통 1분(교통 탭을 보고 있을 때만). 화면이 꺼져 있으면 멈춤
- 받은 데이터는 브라우저 `localStorage`에 저장. 연결이 끊기면 저장본을 "⚠ 연결 실패 — HH:MM 기준 저장본"과 함께 표시
- 먹거리: 위치 권한을 허용하면 현재 위치, 거부하면 숙소 기준 직선거리순. 필터: 아이 메뉴 / 안 매움 / 줄 짧음

## 먹거리 목록 (`foods.json`) 주의
- 좌표는 **근사값**이고, 줄 길이·메뉴는 일반적인 평판 기준입니다. 영업시간과 위치는 출발 전에 구글맵 링크로 한 번 확인하세요.
- 디즈니랜드 식당 3곳의 "아이 메뉴"는 파크 어린이 식사권 사용처 목록(KKday 상품 설명) 기준입니다.
- 맛집·명소 83곳(먹을거리 36 + 가볼곳 47)을 11개 지역(`area`)으로 묶어 보여 줍니다. 위쪽 두 버튼(먹을거리/가볼곳)으로 탭을 나누고, 지도와 지역 버튼의 개수는 선택한 탭 기준입니다. 아이 메뉴·안 매움·줄 짧음 필터는 먹을거리 탭에만 나옵니다.
- `blog`는 2026-11-15에 실제로 열리는 것을 확인한 한국어 후기 주소입니다(브런치, 뽈레, trvlogue, 트립스토어, 여기어때 매거진). 네이버 블로그는 확인 도구에서 막혀 있어 넣지 않았습니다. 일부는 가게 단독 후기가 아니라 그 지역이나 메뉴를 다룬 글이며, 글 제목을 그대로 표시합니다.
- 후기를 못 찾은 곳은 `blog: null`이고, 화면에는 네이버 블로그 검색 링크만 나옵니다.
- 항목 추가 형식:
  ```json
  { "type": "맛집|명소", "name": "", "area": "", "menu": "", "price": "$", "kidMenu": false, "spicy": false, "queue": "짧음|보통|길다", "note": "", "lat": 22.0, "lng": 114.0, "mapQuery": "구글맵 검색어", "blog": "https://... 또는 null", "blogTitle": "글 제목" }
  ```

## 실행
### PC에서 확인
```bash
npx -y http-server@14.1.1 -p 8000 -c-1 .
```
브라우저에서 `http://localhost:8000` 접속. (`index.html`을 더블클릭으로 열면 `foods.json`을 못 읽어 먹거리 탭만 실패합니다.)

### 휴대폰에서 쓰기 (여행 중에는 이 방법)
노트북이 꺼져도 써야 하므로 무료 정적 호스팅에 올리는 것을 권장합니다.
1. GitHub에 새 저장소를 만들고 `index.html`, `foods.json`을 올립니다.
2. 저장소 Settings → Pages → Branch를 `main` / `root`로 지정하고 저장합니다.
3. 몇 분 뒤 `https://<아이디>.github.io/<저장소>/` 주소가 생깁니다.
   - 위치 권한은 https에서만 동작합니다. GitHub Pages는 https라 그대로 됩니다.
   - 저장소를 공개로 하면 주소를 아는 누구나 볼 수 있습니다. 개인정보는 들어 있지 않습니다.

### 홈 화면에 추가
- iPhone(Safari): 공유 버튼 → "홈 화면에 추가"
- Android(Chrome): ⋮ 메뉴 → "홈 화면에 추가"

## 하지 않은 것
- 로그인, 결제, 개인정보 저장 없음
- Google Places 같은 외부 장소 API는 2단계(선택). 키가 필요하므로 넣게 되면 코드에 직접 쓰지 말고 별도 설정으로 분리할 것
- 오프라인 첫 실행(한 번도 데이터를 받은 적 없을 때)은 지원하지 않음. 출발 전에 한 번 열어 두면 저장본이 생깁니다
