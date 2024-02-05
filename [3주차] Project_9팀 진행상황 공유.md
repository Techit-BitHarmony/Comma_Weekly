## 팀 구성원, 개인 별 역할

---

- 조유민[팀장] - 앨범 관련 기능 개발
- 김용욱 - 도네이션 관련 기능 개발
- 김주연 - QA
- 조동국 - ncp video station 관련 기능 검색 및 개발
- 박상혁 - 커뮤니티 관련 기능 개발
- 문창현 - 회원 관련 기능 개발

## 팀 내부 회의 진행 회차 및 일자
1. 1회차(2024.01.30) 디스코드 진행
- PR을 통한 코드 리뷰 및 리펙토링 진행 및 강사님 피드백을 통해 기술적 토의 진행
3. 2회차(2024.02.02) 디스코드 진행
- 주말 전 PR을 통한 코드 리뷰 선 진행 및 기능 관련 회의 진행
---


강사님 피드백
- 현재 커뮤니티 기능에 대해 필요한 인력 재분배
- ncp 기능에 따른 기술적 피드백
- 각 도메인별 피드백


## 현재까지 개발 과정 요약 (최소 500자 이상)
1. 각자 담당한 도메인 개발 진행
   - 각자 담당한 도메인 별로 WBS 및 요구사항 명세서에 따라 개발 진행
   - 이때 개발에서 발생할 것 같은 문제들은 Notion을 통해서 변경사항 알림 페이지를 통해 미리 작성
   - (Global 및 Yml 파일 등 글로벌하게 사용되는 파일들 또는 추후에 병합 시 고려해야 할 문제점 등 )

#### 조유민
- 추천 관련 기능 Entity , Service , Repo 개발 진행
- StreamingCount에 대한 Entity 초안 작성
- Postman을 통한 기능 테스트 중
#### 김용욱
- quartz 스케줄러를 이용해 특정일에 후원하는 기능 개발
    - quartz 라이브러리를 사용하기 위한 설정 추가
    - job, jobDetail, trigger 파일 추가
    - 정기후원을 스케줄러에 등록하는 서비스 레이어 추가
    - 스케줄러가 사용할 데이터를 저장하기 위해 donation regular 엔티티 추가
#### 박상혁
- 결제 기능 TossPaymentsServcie로 분리, 메서드 분리
- Member 필드 연동하여 인증 관련 로직 추가
- Exception 정의 및 리팩토링
- 커뮤니티 글, 댓글 CRUD 기능 구현
#### 조동국
- NCP Action Python Code 수정
    - 인코딩 진행 완료 시 callback 처리
- NCP Cloud function에서 Encoding 상태 callback을 전달 받아 처리 (Redis Pub/Sub)
    - 이벤트 내용 Publish하여 Redis에 전달
    - 클라이언트 측에서 Subscribe를 해두고 Event 발생 시 SEE 방식을 통해 Client에서 인코딩 정보를 받아 볼 수 있도록 구현
#### 문창현
- refresh token을 이용한 access token 재발급 구현
- Redis를 사용하여 refresh token 구현 및 저장
- 로그인 된 사용자의 DB 조회
---
## 개발 과정에서 나왔던 질문 (최소 200자 이상)
- quartz 스케줄러에서 여러 job을 실행시킬 때 어떤 방식으로 작동하는가?
    - Round Robin 알고리즘을 이용해 프로세스를 실행하고 그외에 다른 알고리즘도 사용한다고 함.
- quartz 스케줄러는 job을 메모리에 항상 상주시키는 것인가?
    - 메모리 방식으로 사용할 수도 있고, 데이터베이스에 저장할 수도 있음. 등록된 job의 영속성을 위해 데이터베이스를 사용하는 방식을 선택함.
    - 데이터베이스에 저장하는 경우 저장할 데이터들은 직렬화 가능해야함.
- job, jobDetail, trigger 는 각각 어떤 역할을 하는가?
    - job은 특정시간 혹은 특정주기에 실행시킬 작업. 스케줄러를 사용하는 실질적인 목표
    - jobDetail은 job의 주변 환경을 설정한 뒤 job을 생성. 이 프로젝트에서는 job이 사용할 데이터인 jobDataMap을 정의하고, 그 데이터를 포함한 job을 생성
    - trigger는 job이 실행되는 조건을 정의함. 특정 주기, 특정일을 정의 할 수 있음.
        - 이 프로젝트에서는 특정일에 후원기능이 실행 될 수 있게 cron표현식을 이용해 구현을 하였음
    - 마지막으로 scheduler가 jobDetail과 trigger를 매개변수로 받아서 schedule을 등록함. 스케줄의 수정,삭제 기능을 추가할 예정
    - joblistener를 설정하여 각 job, trigger, schedule이 실행 되기 전, 중, 후에 로그를 출력하거나 추가 로직을 설정할 수 도 있다.
- NCP CloudFunction의 Callback 요청을 받기 위한 방법?
    - localhost 상태로는 ncp에서 보내는 callback을 받을 수 없었기에 서버를 띄우거나 하는 방법이 필요했다.
    - ngrok 서비스를 이용하여 테스트를 진행할 수 있었다.
- Encoding 상태를 전달하기 위해 어떤 방법을 사용할까?
    - Short Polling
        - Request / Response, 흔히 알던 요청 시 응답 방식이다.
    - Long Polling
        - 긴 Connection을 유지하여 한 번의 요청과 여러 번의 응답을 진행한다.
    - Web Socket
        - 실시간 통신, 서버와 클라이언트는 지속적인 양방향 통신을 진행한다.
    - SSE (채택)
        - 양방향 통신을 진행하는 웹소켓과 다르게 서버에서만 전달을 하는 단방향 통신이다.
        - 인코딩 상태 callback 시에만 내용 전달이 필요할 것 같아 채택하게 되었다.
    - 참고
        - [https://surviveasdev.tistory.com/entry/웹소켓-과-SSEServer-Sent-Event-차이점-알아보고-사용해보기](https://surviveasdev.tistory.com/entry/%EC%9B%B9%EC%86%8C%EC%BC%93-%EA%B3%BC-SSEServer-Sent-Event-%EC%B0%A8%EC%9D%B4%EC%A0%90-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B3%A0-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EA%B8%B0)
        - https://gilssang97.tistory.com/69
- Redis Pub/Sub 동작 과정?
    - Redis에서 지원하는 Pub/Sub를 이용하면 위 SSE 처리를 좀 더 수월하게 할 수 있다.
        - 클라이언트는 서버의 API를 통해 채널을 Subscribe한다.
        - 서버는 이벤트(callback)가 발생할 때 마다 채널에 메시지를 Publish 한다.
        - Redis는 메시지가 전달되면 적절하게 처리하여 클라이언트로 전송한다.
- 스트리밍 수는 일정한 스트리밍이 진행된 후 되야하는가? 아니면 클릭하면 올라가야 하는가?
   - 만약 진행이 된 후라면 어떤 방식으로 이를 인지할 수 있을까?
- 앨범 좋아요는 취소가 있을까? 있다면 매일 새롭게 취소하고 좋아요 하면 인기 순위에 반영되는게 아닐까?
   - 그렇다면 어떻게 진행을 해야 하는가?
- 커뮤니티 관련 기능의 분담은 어떻게 해야 하는가?
- 프론트 없이 인증, 권한 관련 테스트 하는 방법
    - 포스트맨 이용하여 헤더에 Authorization 자동으로 추가하여 진행
- 출금 관련하여 관리자가 어떻게 처리할지?
    - 자동 입금 API 있지만 조건이 까다로워 우선 출금 완료 버튼을 클릭하여 처리
- 출금 신청을 수정하는 경우 크레딧 변동을 어떻게 처리할지
    - 수정한 금액과 기존 금액을 비교하여 추가 출금, 반환하는 것으로 처리
- Global Response에 관한 내용
- ncp 협업에 관한 내용
---
## 개발 결과물 공유
---
- Github Repository URL: https://github.com/Techit-BitHarmony/Comma
- Notion : https://www.notion.so/9-4ee8e96c7a2d4c79ba7799897a1f1352
- Trello : https://trello.com/invite/b/E1ayx7TG/ATTI479a7a496bf7cc3fb60944c256cac1803778C523/9th-ideaboard
