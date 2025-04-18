## REST API (2025.04.17)

### 수많은 API들이 REST 방법론을 따르는 이유

#### 1. 대표적인 API

1. 지도
    1. 구글
    2. 네이버
    3. 카카오 등
2. 로그인
    1. 구글
    2. 네이버
    3. 카카오
    4. 애플 등

#### 2. 대표적인 API의 공통점

- 무료
- 제 3의 데이터를 제공
- REST 방법론을 따른 RESTful API
- 작은 데이터를 주고, 더 큰 데이터를 얻음
    - 예시
        - 지도 API (구글, 네이버, 카카오 등)
            - 초기에는 사용자에게 일방적으로 데이터를 제공
            - 점차 사용자가 늘어남
            - 새로운 건물이 생기거나, 새로운 가게가 생기면 그 사장들이 해당 지도 API에 본인들의 건물을 등록하고 싶어함
            - 지도 API의 데이터가 풍성해
        - 로그인 API (카카오, 애플 등)
            - 타 사업자들에게 본인들이 가진 사용자 정보를 제공
            - 그 사용자의 행동 패턴, 소비 패턴 등의 추가적인 데이터를 얻을 수 있음
            - 이 데이터들을 기반으로 광고 등을 함

#### 3. RESTful API의 이점

1. 확장성
    1. 본인들이 만든 API를 사용하는 사람들(개발자)이 본인들의 서버 상태를 기억할 필요가 없음
    2. 본인들의 API 사용자가 많아지면, 서버 여러 개 띄우면 됨
2. 표준화
    1. 본인들이 만든 API를 사용하는 사람들(개발자)이 어떤 언어, 어떤 프레임워크를 사용하는지 상관 없이 이용 가능
3. 유지보수성, 성능 최적화
    1. url 요청 나누기, 합치기 등을 손쉽게 할 수 있기 때문에 유지보수성과 성능 최적화에 유리