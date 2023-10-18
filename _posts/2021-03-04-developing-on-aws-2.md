---
title: Developing on AWS 2일차
author: kjb4494
date: 2021-03-04 15:00:00 +0900
categories: [Cloud Platform, AWS]
tags: [cloud, aws]
---

### [_DynamoDB_](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Introduction.html)

- [_DDB 용어_](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/HowItWorks.CoreComponents.html)

  1. Table(테이블)
  1. Item(항목)
  1. PK -> partition key(hash key) + sort key(range key)
     - PK Best Paractice
       하나의 pk에 데이터가 쏠리면 한 파티션의 부담이 커진다. 반드시 그런 key를 사용해야 한다면 뒤에 랜덤값을 붙여 파티션을 쪼개주는게 효율적이다.
     - sort key
  1. Attribute(속성)

- DDB의 특징

  - serverless 서비스이다.
  - 기본적으로 삼중화 저장을 한다. (데이터를 3벌 복제)
  - 데이터 저장 공간의 제약이 없다.
  - RCU, WCU 설정으로 DDB의 성능을 결정한다.
  - 전용 캐시(DAX)가 있다.
  - 그럼 삼중화 분산환경에서 일관성을 어떻게 확보하는가? - 중요!

    > - [_CAP 이론_](https://itwiki.kr/w/CAP_%EC%9D%B4%EB%A1%A0)
    >   C(일관성): 최신 값 응답을 중시
    >   A(가용성): 에러를 내지않는 것을 중시
    >   P(분산환경): 무조건적인 선택사항...

    DDB는 default가 최종 일관성(AP), 선택적으로 강력한 일관성(CP) 가능

- 모범 사례
  - 균일한 워크로드
    읽기 및 쓰기 작업을 파티션 키 전체에 균등하게 분산하여 처리량을 극대화한다.
  - 핫 데이터와 콜드 데이터
    자주 액세스하는 데이터(핫)를 자주 액세스하지 않는 데이터(콜드)와 분리
  - DAX 사용
    DDB는 DAX라는 전용캐시공간이 있기 때문에 redis 등을 사용할 때처럼 key missing에 대한 처리 로직이 필요없다.
  - 배치 작업의 오류 처리
    지수 백오프 알고리즘을 사용하여 배치 작업에서 실패한 테이블과 항목을 재시도한다.
  - 처리량 예외 처리(ProcisionedThroughtputExeededException)

### lambda

- 서버리스 컴퓨팅

  1. 프로비저닝하거나 관리할 서버가 없음
  1. 사용량에 따른 확장
  1. 유휴 상태에 대해 비용을 지불할 필요가 없음
  1. 내장된 고가용성 및 내결함성

- 람다 특징
  - 람다는 이벤트 소스가 호출 해주지 않으면 함수가 되지않는다.
  - AWS 서비스에 접근하기 위해 IAM을 통과하려면 role을 씌워준다.
  - 한 번의 이벤트에서 최대 실행 시간이 15분이며, 이 시간을 넘겨야하는 프로그램엔 적합하지않다.
  - 핸들러는 어떤 이벤트를 통해 실행할지 정하는 엔트리포인트다.
  - 이벤트는 로직을 처리하는데 필요한 파라미터를 전달한다.
  - 배포하는 순간 람다의 고유한 ARN이 발행된다.
  - 람다 호출 시 각각의 요청에 request ID가 붙는다.
  - 서버리스이기 때문에 로컬의 영구적인 디스크 공간을 쓸 수 없다. -> 클라우드 저장공간을 사용할 것
  - 기본적으로 로그 쓰기 권한에 대한 role을 가지고 있다. (LambdabasicExcution)
  - 가급적 프록시 통합 기능 사용
- [_python에서의 handler_](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/python-handler.html)
  함수 핸들러는 임의의 이름일 수 있지만 Lambda 콘솔의 기본값은 lambda_function.lambda_handler입니다. 이 이름은 함수 이름을 lambda_handler로 나타내고 핸들러 코드가 저장된 파일은 lambda_function.py에 저장됩니다.

### AWS SAM

서버리스 앱을 정의하기 위한 템플릿 기반 배포 모델

### AWS Step Functions

람다를 순서대로 실행해야할 때 쓰는 서비스

- 어떤 작업에 사용하는지?
  - 실패한 작업을 재시도할 때
  - 작업을 순서대로 배열할 때
  - try/catch/finally를 사용할 때
  - 데이터에 근거하여 선택하고 싶을 때
  - 작업을 병렬로 실행하고 싶을 때
