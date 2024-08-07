# 3주차 정리
- 목차
  - 웹 크롤러 설계
  - 채팅 시스템 설계
    
## 웹 크롤러 설계
- 검색 엔진에서 널리 쓰는 기술
- 웹에 새로 올라오거나 갱신된 콘텐츠를 찾아내는 것이 주된 목적
- 몇 개 웹 페이지에서 시작하여 그 링크를 따라 나가면서 새로운 콘텐츠를 수집함
  - 검색 엔진 인덱싱
    - 크롤러는 웹 페이지를 모아 검색 엔진을 위한 로컬 인덱스를 만듦 ,ex) 구글 검색 엔진
  - 웹 아카이빙
    - 나중에 사용할 목적으로 장기보관하기 위해 웹에서 정보를 모으는 절차
  - 웹 마이닝
  - 웹 모니터링
    - 크롤러를 사용해 인터넷에서 저작권이나 상표권이 침해되는 사례를 모니터링 할 수 있음

### 1.문제 이해 및 설계 범위 확정
- 웹 크롤러의 기본 알고리즘
  1. URL 집합 입력이 주어지면, 해당 URL들이 가리키는 모든 웹 페이지를 다운로드함
  2. 다운받은 웹 페이지에 URL들을 추출함
  3. 추출된 URL들을 다운로들할 URL 목록에 추가하고, 위의 과정을 처음부터 반복함
 (실제는 더욱 복잡하지만... 대략적으로)

- 웹 크롤러가 만족 시켜야 할 속성
  - 규모 확장성 : 병행성을 활용
  - 안정성 : 비정상적인 입력이나, 환경에 잘 대응해야함
  - 예절(politeness) : 크롤러는 수집 대상 웹 사이트에 짧은 시간 동안 너무 많은 요청을 보내면 안됨
  - 확장성 : 새로운 형태의 콘텐츠를 지원하기 쉬워야함

### 2.개략적 설계안 제시 및 동의 구하기

 <img width="691" alt="스크린샷 2024-07-29 오후 12 27 06" src="https://github.com/user-attachments/assets/5726c614-b1d1-488d-88a3-83965afb7e6f">

- 시작 URL 집합
  - 웹 크롤러가 크롤링을 시작하는 출발점
  - 크롤러가 가능한 많은 링크를 탐색할 수 있도록 하는 URL을 고르는것이 바람직함
  - 주제별로 세분화된 다른 URL을 사용함
  - 등등...
- 미수집 URL 저장소
  - 웹 크롤러는 크롤링 상태를 다운로드할 URL 과 다운로드된 URL로 나눔
  - 다운로들할 URL을 저장 관리하는 컴포넌트를 미수집 URL 저장소라고 부름 (FIFO,Queue)
- HTML 다운로더
  - 인터넷에서 웹 페이지를 다운로드하는 컴포넌트
  - 다운로드할 페이지의 URL은 미수집 URL저장소가 제공함
- 도메인 이름 변환기
  - 웹 페이지를 다운받으려면, URL을 IP 주소로 변환하는 절차가 필요함
  - HTML 다운로더는 도메인 이름 변환기를 사용하여 URL에 대응되는 IP주소를 알아냄
- 콘텐츠 파서
  - 웹 페이지를 다운로드하면 파싱과 검증 절차를 거쳐야 함
  - 크롤링 서버 안에 콘텐츠 파서를 구현하면 크롤링 과정이 느려지기때문에, 독립된 컴포넌트로 만듦
- 중복 콘텐츠
  - 데이터 중복을 줄이고, 데이터 처리에 소용되는 시간을 줄임
  - 웹 페이지의 해시 값을 비교하는 방법 사용
- URL 추출기
  - HTML 페이지를 파싱하여 링크들을 골라내는 역할을 함
- URL 필터
  - 특정한 콘텐츠 타입이나 파일 확장자를 갖는 URL, 접속시 오류 생기는 URL, deny list에 포함된 URL 등을 크롤링 대상에서 배제함
- 웹 크롤러 작업 흐름
<img width="823" alt="스크린샷 2024-07-29 오후 12 47 29" src="https://github.com/user-attachments/assets/6c2fcf59-68e7-42b9-8b5e-87b98fae1a0f">

### 3.상세 설계

- DFS vs BFS
  - 웹은 directed graph, 페이지는 node, 하이퍼링크는 edge 라고 보면됨
  - 크롤링 프로세스는 이런 그래프를 탐색하는 과정
  - 그래프 크기가 클 경우, 어느 정도로 깊숙이 가게 될지 가늠이 어렵기때문에, DFS는 좋은 선택이 아닐 수 있음
  - 주로 BFS(FIFO 큐 알고리즘)를 사용함
  - 이 알고리즘도 문제가 발생함
    - 한 페이지에서 나오는 링크의 상당수는 같은 서버로 되돌아감 -> 이때 이 링크들을 병렬로 처리하게되면 서버는 수많은 요청으로 과부하 걸림(예의 없는 크롤러)
    - 표준적 BFS 알고리즘은 URL간에 우선순위를 두지 않음 -> 우선순위를 구별하는것이 필요함
- 미수집 URL 저장소
  - 미수집 URL 저장소를 활용하면 예의를 갖춘 크롤러, URL사이의 우선순위와 신선도를 구별하는 크롤러를 구현할 수 있음
  - 예의
    - 웹 크롤러는 수집 대상 서버로 짧은 시간 안에 너무 많은 요청을 보내는 것(impolite)을 삼가해야 함
    - 예의 바른 크롤러는 동일 웹 사이트에 대해서는 한 번에 한 페이지만 요청하는 것
    - 같은 웹 사이트의 페이지를 다운받는 테스크는 시간차를 두고 실행하도록 하면 됨
    - 이 요구사항을 위해서 웹 사이트의 호스트명과 다운로드를 수행하는 작업 스레드 사이의 관계를 유지하면 됨
      - 각 다운로드 스레드는 별도 FIFO 큐를 가지고 있어서, 해당 큐에서 꺼낸 URL만 다운로드하면 됨
<img width="772" alt="스크린샷 2024-07-29 오후 1 16 21" src="https://github.com/user-attachments/assets/f04386fd-71e6-4697-9472-5ad109020935">

    - 큐 라우터 : 같은 호스트에 속한 URL은 언제나 같은 큐로 가도록 보장함
    - 매핑 테이블 : 호스트 이름과 큐 사이의 관계를 보관하는 테이블
    - FIFO 큐 : 같은 호스트에 속한 URL은 언제나 같은 큐에 보관됨
    - 큐 선택기 : 큐들을 순회하면서 큐에서 URL을 꺼내서 해당 큐에서 나온 URL을 다운로드하도록 지정된 작업 스레드에 전달하는 역할
    - 작업 스레드 : 전달된 URL을 다운로드하는 작업 수행
   
- 우선순위
  - 크롤러 입장에서는 중요한 페이지 먼저 수집하도록 하는 것이 바람직함
  - 다양한 척도 사용 : 페이지랭크, 트래픽 양, 갱신 빈도...
  - 우선순위 계산을 위해 '순위결정장치'를 사용함 -> 큐 선택기에서 순위가 높은 큐에서 더 자주 꺼내지도록 설계
  
<img width="929" alt="스크린샷 2024-07-29 오후 1 27 22" src="https://github.com/user-attachments/assets/b5700bb8-0269-4399-b8de-6d19e94e16aa">

<img width="857" alt="스크린샷 2024-07-29 오후 1 27 30" src="https://github.com/user-attachments/assets/4a728a81-9ac4-4c8f-b362-ed25668c28a0">

- 전면 큐 : 우선순위 결정 과정을 처리함
- 후면 큐 : 크롤러가 예의 바르게 동작하도록 보증함
- 데이터의 신선함을 유지하기 위해서는 이미 다운로드한 페이지라도, 주기적으로 재수집을 해줘야함
  - 웹 페이지의 변경 이력 활용
  - 우선순위를 활용해, 중요한 페이지는 좀 더 자주 재수집

- 미수집 URL 저장소를 위한 지속성 저장장치
  - 처리해야할 URL을 모두 메모리에 보관하는것은 바람직하지 않음
  - 대부분의 URL은 디스크에 두지만, IO 비용을 줄이기 위해 메모리 버퍼에 큐를 두는 것임
  - 버퍼가 주기적으로 데이터를 디스크에 기록할 것임

- HTML 다운로더
  - HTTP 프로토콜을 통해 웹 페이지를 내려받음
  - Robots.txt(로봇 제외 프로토콜)
    - 웹 사이트가 크롤러와 소통하는 표준적 방법
    - 크롤러가 수집해도 되는 페이지 목록을 바탕으로 긁어갈 파일들의 규칙을 확인함
  - HTML 다운로더 성능 최적화
    1. 분산 크롤링
       - 성능을 높이기 위해 크롤링 작업을 여러 서버에 분산함
       - URL 공간을 작은 단위로 분할하여, 각 서버는 그중 일부의 다운로드를 담당하도록 함
    2. 도메인 이름 변환 결과 캐시
       - DNS 요청을 보내고 결과를 받는 작업의 동기적 특성이 있음
       - 결과를 받기 전까지는 다음 작업을 진행할 수 없음
       - 따라서, DNS 조회 결과로 얻어진 도메인 이름과 IP 주소 사이의 관계를 캐시에 보관해 놓고, cron job을 돌려 주기적으로 갱신하도록 설계
    3. 지역성
       - 크롤링 작업을 수행하는 서버를 지역별로 분산하는 방법 
    4. 짧은 타임아웃
       - 응답이 느리거나, 응답이 없을때, 대기시간의 최대 시간을 미리 정해두는거
       - 이 시간 동안 서버가 응답하지 않으면, 크롤러는 해당 페이지 다운로드를 중단하고 다음 페이지로 넘어감
    - 안정성
      - 안정 해시 : 다운로더 서버들에 부하를 분산할 때 적용 가능한 기술
      - 크롤링 상태 및 수집 데이터 저장 : 장애가 발생한 경우에도 쉽게 복구할 수 있도록 크롤링 상태와 수집된 데이터를 지속적 저장장치에 기록해둬야함
      - 예외 처리
      - 데이터 검증 : 시스템 오류를 방지하기 위한 중요한 수단
    - 확장성
      - 확장성을 갖도록 설계해야 함
      <img width="834" alt="스크린샷 2024-07-29 오후 1 47 35" src="https://github.com/user-attachments/assets/38c622b7-d263-41dd-82b4-5043112bd802">

- 문제 있는 콘텐츠 감지 및 회피
  1. 중복 콘텐츠
     - 해시나 체크섬을 사용하여 탐지 가능함
  2. 거미 덫
     - 크롤러를 무한 루프에 빠뜨리도록 설계한 웹 페이지
     - 알고리즘을 만들거나, 수작업으로 필터를 해줘서 해결하는 방법이 있음
  3. 데이터 노이즈
     - 가치가 없는 페이지는 제외해야 함
    
### 마무리
- 면접관과 추가로 논의해보면 좋은거..
  - 서버 측 렌더링
  - 원치 않는 페이지 필터링
  - 데이터베이스 다중화 및 샤딩 다중화
  - 수평적 규모 확장성
  - 가용성,일관성,안전성
  - 데이터 분석 솔루션


## 채팅 시스템 설계


### 1. 문제 이해 및 설계 범위 확정
- 면접관이 원하는 앱이 정확히 무엇인지 알아내야함
- 1:1 채팅인지, 그룹 채팅인지, 채팅 이력은 얼마나 보관할지...

### 2. 개략적 설계안 제시 및 동의 구하기
- 채팅 서비스가 필요한 기능
  - 클라이언트로부터 메시지 수신
  - 메시지 수신자 결정 및 전달
  - 수신자가 접속 상태가 아닐때, 수신자가 접속할 때까지 해당 메시지 보관
    <img width="754" alt="스크린샷 2024-07-29 오후 2 09 15" src="https://github.com/user-attachments/assets/709e0127-5ae6-4527-a82d-d8ddef279f88">

  - 어떤 프로토콜을 사용할지 결정해야함
  - 송신자가 수신자에게 메시지를 전달할 때, HTTP프로토콜을 사용함
  - 채팅 서비스와의 접속에는 keep-alive 헤더를 사용하면 효율적임 (서버와 클라이언트 사이의 연결을 끊지 않고 계속 유지할 수 있음)
  - TCP 접속 과정에서 발생하는 핸드셰이크 횟수도 줄일 수 있음
- HTTP는 클라이언트가 연결을 만드는 프로토콜이며, 서버에서 클라이언트로 임의 시점에 메시지를 보내는 데는 쉽게 쓰일 수 없음
  - 서버가 연결을 만드는 방법들이 있음
    - 폴링
    - 롱폴링
    - 웹소켓 ...
- 폴링
  - 클라이언트가 주기적으로 서버에게 새 메시지가 있느냐고 물어보는 방법
  - 폴링을 자주하면 비용이 증가함
  <img width="792" alt="스크린샷 2024-07-29 오후 2 21 27" src="https://github.com/user-attachments/assets/ce4b9417-63d9-4fa5-a7ec-adfc837bbedf">
- 롱 폴링
  - 클라이언트는 새 메시지가 반환되거나 타임아웃 될 때까지 연결을 유지함
  - 클라이언트는 새 메시지를 받으면 기존 연결을 종료하고 서버에 새로운 요청을 보내어 모든 절차를 다시 시작함
  - 메시지를 보내는 클라이언트와 수신하는 클라이언트가 같은 채팅 서버에 접속 안될 수도 있음
    - HTTP 서버들은 보통 무상태 서버이기에,  로드밸런싱을 위해 라운드 로빈 알고리즘을 사용하는 경우, 메시지를 받은 서버는 해당 메시지를 수신할 클라이언트와
    - 롱 폴링 연결을 가지고 있지 않은 서버일 수 있음
  - 서버 입장에서는 클라이언트가 연결을 했는지 알 수가 없음
  <img width="811" alt="스크린샷 2024-07-29 오후 2 28 23" src="https://github.com/user-attachments/assets/0fa388a3-2560-486d-8398-69808c11a6e3">

    
- 웹 소켓
  - 서버가 클라이언트에게 비동기 메시지를 보낼 때 가장 널리 사용하는 기술
  <img width="807" alt="스크린샷 2024-07-29 오후 2 28 30" src="https://github.com/user-attachments/assets/2e824741-83c0-4ad2-b25f-7e601d9eec75">

  - 한번 맺어진 연결은 항구적(연결이 지속적이고, 안정적)이며 양방향
  - 처음에는 HTTP 연결이지만, 특정 핸드셰이크 절차를 거쳐 웹소켓 연결로 업그레이드됨
  - 이 항구적 연결이 만들어지면, 서버는 클라이언트에게  비동기적으로 메시지를 전송할 수 있음
  - 웹 소켓은 HTTP,HTTPS가 쓰는 프로토콜인 80이나 443을 기본 포트번호로 그대로 쓰기에 방화벽이 있는 곳에서도 잘 작동함
  <img width="658" alt="스크린샷 2024-07-29 오후 2 41 32" src="https://github.com/user-attachments/assets/1b17b5bc-456c-485f-a598-1464b3fcc43e">

  - 웹 소켓을 이용하면 메시지를 보낼 때나 받을 때 동일한 프로토콜을 사용할 수 있음
  - 설계와 구현도 단순하고 직관적임

- 개략적 설계안
<img width="703" alt="스크린샷 2024-07-29 오후 2 44 02" src="https://github.com/user-attachments/assets/1f120f79-a7e9-4eeb-bebe-d97cc734c293">

- 무상태 서비스
  - 로그인,회원가입,사용자 프로파일 등... 전통적인 요청/응답 서비스
- 상태 유지 서비스
  - 클라이언트가 채팅 서버와 독립적인 네트워크 연결을 유지해야 하기 때문
- 제3자 서비스 연동
  - 푸시 알림
- 규모 확장성
<img width="842" alt="스크린샷 2024-07-29 오후 2 49 07" src="https://github.com/user-attachments/assets/d8ee80a2-eea9-4771-8e1f-7ea0f8da41fa">

- 채팅 서버는 클라이언트 사이에 메시지를 중계하는 역할을 담당
- 접속상태 서버는 사용자의 접속 여부를 관리
- API 서버는 로그인, 회원가입,프로파일 변경 등.. 나머지 전부를 처리
- k-v 저장소는 채팅 이력을 보관함

- 저장소
  - 채팅 시스템이 다루는 데이터는 보통 두 가지
    1. 친구 목록, 사용자 프로필 등.. 일반적 데이터 -> SQL사용
    2. 채팅 시스템 고유한 데이터 (채팅 이력)
       - 양이 엄청남
       - 최근에 주고받은 메시지
       - 검색을 통해 언급된 메시지, 무작위 데이터 접근 (점프)
       - 읽기와 쓰기의 비율이 1:1임
       - kev-value 저장소를 사용 (수평적 규모 확장)
  - kev -value 저장소
    - 접근 지연시간이 낮음
    - 많은 채팅 시스템이 이런 저장소를 채택함
- 데이터 모델
  - 메시지 데이터를 어떻게 보관할지...
  - 1:1 채팅을 위한 메시지 테이블
  - 그룹 채팅을 위한 메시지 테이블
  - 메시지 ID
    - 메시지들의 순서도 표현할 수 있어야 함
    - message_id의 값은 유니크해야함
    - ID값은 정렬 가능해야 하며, 시간 순서와 일치해야함
    - 이 두 조건을 만족시키기 위해서는 RDBMS의 auto_increment가 있지만, NoSQL은 없음
    - 스노플레이크 같은 순서 번호 생성기 이용
    - 지역적 순서(같은 그룹 안에서만 보증하면 충분함..) 번호 생성기 이용
       
### 3. 상세 설계
- 서비스 탐색
  - 클라이언트에게 가장 적합한 채팅 서버를 추천하는 역할을 함
  - 기준은 클라이언트의 위치, 서버의 용량 등...
  - ex) 아파치 주키퍼가 있음
  
<img width="764" alt="스크린샷 2024-07-29 오후 3 20 16" src="https://github.com/user-attachments/assets/0ab0ea6f-bd6d-4d81-92d1-05de8a0f511d">

  1. 사용자 A가 시스템에 로그인 시도함
  2. 로드밸런서가 로그인 요청을 API 서버들 가운데 하나로 보냄
  3. API 서버가 사용자 인증을 처리하고 나면 서비스 탐색 기능이 최적의 채팅 서버를 찾음

- 메시지 흐름
  - 1:1 채팅 메시지 처리 흐름
<img width="825" alt="스크린샷 2024-07-29 오후 3 25 05" src="https://github.com/user-attachments/assets/dcc79fd7-2104-4fb3-a368-55690fd1368f">

- 여러 단말 사이의 메시지 동기화
<img width="810" alt="스크린샷 2024-07-29 오후 3 29 35" src="https://github.com/user-attachments/assets/39ebfedd-b9ea-415c-8e6d-29724c7e3a45">

  - 각 단말은 동일한 변수를 유지하는데, 해당 단말에서 관측된 가장 최신 메시지의 ID를 추적하는 용도임
    - 수신자 ID가 현재 로그인한 사용자 ID와 같음
    - k-v 저장소에 보관된 메시지로, 그 ID가 위에서 정한 변수보다 큼

  - 소규모 그룹 채팅에서의 메시지 흐름
<img width="762" alt="스크린샷 2024-07-29 오후 3 29 44" src="https://github.com/user-attachments/assets/6945e61a-2b91-4f63-9926-bee80ee496d3">

  - A가 보낸 메시지는 사용자 B와C의 메시지 동기화 큐에 복사됨
  - 이 큐를 사용자 각각에  할당된 메시지 수신함으로 생각하면됨
  - 소규모 적합한 이유
    - 새로운 메시지 확인을 위해, 자기 큐만 보면됨
    - 그룹이 크지 않기에, 수신자별로 큐에 넣는 작업의 비용이 문제가 되지 않음
    - 많은 사용자를 지원해야 한다면, 이런 방법이 바람직하지 않음
      - 한 수신자는 여러 사용자로부터 오는 메시지를 수신할 수 있어야 함
  <img width="721" alt="스크린샷 2024-07-29 오후 3 35 11" src="https://github.com/user-attachments/assets/543a9622-5e4f-4439-b5b2-6daa47327c35">

- 접속상태 표시
  - 접속상태 서버를 통해 사용자의 상태를 관리함

  - 사용자 로그인
    - 클라이언트와 실시간 서비스 사이에 웹소켓 연결을 통해, 클라이언트 상태와 타임스탬프 값을 k-v 저장소에 보관함
  - 로그아웃
    - k-v 저자소에 보관된 사용자 상태가 온라인 -> 오프라인으로 바뀜
  - 접속장애
    - 인터넷 연결이 끊어지면, 웹소켓 같은 지속성 연결도 끊어짐
    - 가장 좋은 방법은 박동 검사를 통해 문제를 해결하면됨
    - 주기적으로 박동 이벤트를 접속상태 서버로 보내고, 주기적으로 계속 확인을 함
  - 상태 정보의 전송
    - 발행-구독 모델을 사용함
      - 각각의 친구관계마다 채널을 하나씩 두는것
      - 그룹의 크기가 작을때 효과적임
  <img width="806" alt="스크린샷 2024-07-29 오후 5 53 03" src="https://github.com/user-attachments/assets/09b57e0b-5251-489d-916c-fe3729502681">

  - 친구 관계에 있는 사용자가 상태정보 변화를 쉽게 통지 받을 수 있음

### 4. 마무리 
- 면접관과 추가로 논의할 것
  - 채팅 앱을 확장하여 사진이나 비디오 등의 미디어를 지원하도록 하는 방법
  - 종단 간 암호화 : 메시지 발신인과 수신자 이외에는 아무도 메시지 내용을 볼 수 없다는 뜻
  - 캐시 : 클라이언트에 이미 읽은 메시지를 캐시해 두면 서버와 주고받는 데이터 양을 줄일 수 있음
  - 로딩 속도 개선 : 지역적으로 분산하는 네트워크 구축을 통해 앱 로딩 속도를 개선함
  - 오류 처리 : 서버 재접속, 메시지 안정적 전송을 보장하기 위한 방법 등...
