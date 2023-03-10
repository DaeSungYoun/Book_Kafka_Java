# 3. 카프카 기본 개념 설명
## 3.1 카프카 브로커, 클러스터, 주키퍼
* 카프카 브로커는 카프카 클라이언트와 데이터를 주고받기 위해 사용하는 주체이자 데이터를 분산 저장하여 장애가 발생하더라도 안전하게 사용할 수 있도록 도와주는 애플리케이션이다
* ## 데이터 저장, 전송
    * 카프카는 페이지 캐시를 사용하여 디스크 입출력 속도를 높임
    * 페이지 캐시란 OS에서 파일 입출력의 성능 향상을 위해 만들어 놓은 메모리 영역을 뜻한다.
    * 한번 읽은 파일의 내용은 메모리의 페이지 캐시 영역에 저장 시킨다.
    * 추후 동일한 파일의 접근이 일어나면 디스크에서 읽지 않고 메모리에서 직접 읽는 방식
    * 이러한 특징 떄문에 카프카 브로커를 실행하는데 힙 메모리 사이즈를 크게 설정할 필요가 없다
* ## 데이터 복제, 싱크
    * 데이터 복제(replication)는 카프카를 장애 허용 시스템으로 동작하도록 하는 원동력이다
    * 복제의 이유는 클러스터로 묶인 브로커 중 일부에 장애가 발생하더라도 데이터를 유실하지 않고 안전하게 사용하기 위함이다.
    * 카프카의 데이터 복제는 파티션 단위로 이루어진다.
    * 토픽을 생성할 때 파티션의 복제 개수도 같이 설정되는데 직접 옵션을 선택하지 않으면 브로커에 설정된 옵션 값을 따라간다.
    * 프로듀서 또는 컨슈머와 직접 통신하는 파티션을 `리더`, 나머지 복제 데이터를 가지고 있는 파티션을 `팔로워`라고 부른다.
    * 팔로워 파티션들은 리더 파티션의 오프셋을 확인하여 현재 자신이 가지고 있는 오프셋과 차이가 나는 경우 리더 파티션으로부터 데이터를 가져와서 자신의 파티션에 저장하는데, 이 과정을 복제(replication)라고 부른다.
* ## 컨트롤러
    * 클러스터의 다수 브로커 중 한 대가 컨트롤러의 역할을 한다.
    * 컨트롤러는 `다른 브로커들의 상태를 체크하고 브로커가 클러스터에서 빠지는 경우 해당 브로커에 존재하는 리더 파티션을 재분배`한다.
    * 만약 컨트롤러 역할을 하는 브로커에 장애가 생기면 다른 브로커가 컨트롤러 역할을 한다.
* ## 데이터 삭제
    * 카프카는 다른 메시징 플랫폼과 다르게 컨슈머가 데이터를 가져가더라도 토픽의 데이터는 삭제되지 않는다
    * 또한 컨슈머나 프로듀서가 데이터 삭제를 요청할 수도 없다.
    * 오직 브로커만이 데이터를 삭제할 수 있다. 데이터 삭제는 파일 단위로 이루어 지는데 이 단위를 `로그 세그먼트`라고 부른다.
    * 이 세그먼트에는 다수의 데이터가 들어 있기 때문에 일반적인 데이터베이스처럼 특정 데이터를 선별해서 삭제할 수 없다.
    * 세그먼트는 데이터가 쌓이는 동안 파일 시스템으로 열려있으며 카프카 브로커에 `log.segement.btyes` 또는 `log.segment.ms`옵션에 값이 설정되면 세그먼트 팡ㄹ이 닫힌다.
    * 세그먼트 파일이 닫히게 되는 기본값은 1GB용량에 도달했을 때인데 간격을 더 줄이고 싶다면 작은 용량으로 설정하면 된다.
    * 그러나 너무 작은 용량으로 설정하면 데이터들을 저장하는 동안 세그먼트 파일을 자주 여닫음으로써 부하가 발생할 수 있으므로 주의해야 한다.
    * 닫힌 세그먼트 파일은 `log.retention.bytes` 또는 `log.retention.ms` 옵션에 설정값이 넘으면 삭제된다.
    * 닫힌 세그먼트 파일을 체크하는 간격은 카프카 브로커의 옵션에 설정된 `log.retention.check.interval.ms`에 따른다.
* ## 컨슈머 오프셋 저장
    * 컨슈머 그룹은 토픽이 특정 파티션으로부터 데이터를 가져가서 처리하고 이 파티션의 어느 레코드까지 가져갔는지 확인하기 위해 오프셋을 커밋한다.
    * 커밋한 오프셋은 `_consumer_offsets` 토픽에 저장
    * 여기에 저장된 오프셋을 토대로 컨슈머 그룹은 다음 레코드를 가서 처리한다
* ## 코디네이터(coordinator)
    * 클러스터의 다수 브로커 중 한 대는 코디네이터의 역할을 수행한다.
    * 코디네이터는 `컨슈머 그룹의 상태를 체크하고 파티션을 컨슈머와 매칭되도록 분배하는 역할`을 한다
    * 컨슈머가 컨슈머 그룹에서 빠지면 매칭되지 않은 파티션을 정상 동작하는 컨슈머로 할당하여 끊임없이 데이터가 처리되도록 도와준다.
    * 이렇게 파티션을 컨슈머로 재할당하는 과정을 `리밸런스`라고 부른다.
* ## 주키퍼
    * 주키퍼는 카프카의 메타데이터를 관리하는 데에 사용된다.
        ```bash
        bin/zookeeper-shell.sh my-kafka:2181
        ```
    > 주키퍼에서 다수의 카프카 클러스터를 사용하는 방법<br><br>
    주키퍼의 서로 다른 znode에 카프카 클러스터들을 설정하면 된다.<br>
    znode란 주키퍼에서 사용하는 데이터 저장 단위이다. 마치 파일 시스템처럼 znode 간에 계층 구조를 가진다.<br>
    1개의 znode에는 n개의 하위 znode가 존재하고 계속해서 tree 구조로 znode가 존재할 수 있다.<br>
    2개 이상의 카프카 클러스터를 구축할 때는 root znode(최상위 znode)가 아닌 한 단계 아래의 znode를 카프카 브로커 옵션으로 지정하도록 한다.<br>
    각기 다른 하위 znode로 설정된 서로 다른 카프카 클러스터는 각 클러스터의 데이터에 영향을 미치지 않고 정상 동작한다.
## 3.2 토픽과 파티션
* 토픽은 카프카에서 데이터를 구분하기 위해 사용하는 단위이다.
* 토픽은 1개 이상의 파티션을 소유하고 있다. 파티션에는 프로듀서가 보낸 데이터들이 들어가 저장되는데 이 데이터를 `레코드`라고 부른다
* 파티션은 카프카의 병렬처리의 핵심으로써 그룹으로 묶인 컨슈머들이 레코드를 병렬로 처리할 수 있도록 매칭된다.
* 컨슈머의 처리량이 한정된 상황에서 많은 레코드를 병렬로 처리하는 가장 좋은 방법은 컨슈머의 개수를 늘려 스케일 아웃하는 것이다.
* 컨슈머 개수를 늘림과 동시에 파티션 개수도 늘리면 처리량이 증가하는 효과를 볼 수 있다.
* ## 토픽 이름 제약 조건
    * 빈 문자열 토픽 이름은 지원하지 않는다.
    * 토픽 이름은 마침표 하나(.) 또는 마침표 둘(..)로 생성될 수 없다.
    * 토픽 이름의 길이는 249자 미만으로 생성되어야 한다.
    * 토픽 이름은 영어 대소문자와 숫자 0부터 9 그리고 마침표(.), 언더바(_), 하이픈(-) 조합으로 새엇ㅇ할 수 있다. 이외의 문자열이 포함된 토픽 이름은 생성 불가하다.
    * 카프카 내부 로직 관리 목적으로 사용되는 2개 토픽(_consumer_offsets, _transaction_state)과 동일한 이름으로 생성 불가능 하다.
    * 카프카 내부적으로 사용하는 로직 때문에 토픽 이름에 마침표(.)와 언더바( _ )가 동시에 들어가면 안 된다. 생성은 할 수 있지만 사용 시 이슈가 발생할 수 있기 때문에 마침표(.)와 언더바( _ )가 들어간 토픽 이름을 사용하면 WARNING 메시지가 발생한다.
    * 이미 생성된 토픽 이름의 마침표(.)를 언더바(_ )로 바꾸거나 언더바(_ )를 마침표(.)로 바꾼 경우 신규 토픽 이름과 동일하다면 생성할 수 없다. 예를들어, to.pic 이름의 토픽이 생성되어 있다면 to_pic 이름의 토픽을 생성할 수 없다.
* ## 의미 있는 토픽 이름 작명 방법
    * 토픽 이름을 통해 어떤 개발환경에서 사용되는 것인지 판단 가능해야 하고 어떤 애플리케이션에서 어떤 데이터 타입으로 사용되는지 유추할 수 있어야 한다.
    * 휴먼에러로 인한 실수를 방지하기 위해 대문자와 소문자를 섞어서 쓰는 카멜케이스를 사용하기보다는 케밥케이스(kebab-case) 또는 스네이크 표기법(snake_case)과 같이 소문자를 쓰되 구분자로 특수문자를 조합하여 사용하면 좋다.
        > 토픽 작명의 템플릿과 예시<br><br>
        <환경>.<팀-명>.<애플리케이션-명>.<메시지-타입><br>
        예시) prd.marketing-team.sms-platform.json<br><br>
        <프로젝트-명>.<서비스-명>.<환경>.<이벤트-명><br>
        예시) commerce.payment.prd.notification<br><br>
        <환경>.<서비스-명>.<JIRA-번호>.<메시지-타입><br>
        예시) dev.email-sender.jira-1234.email-vo-custom<br><br>
        <카프카-클러스터-명>.<환경>.<서비스-명>.<메시지-타입><br>
        예시) aws-kafka.live.marketing-platform.json
## 3.3 레코드
* 레코드는 `타입스탬프`, `메시지 키`, `메시지 값`, `오프셋`, `헤더`로 구성되어 있다
* 프로듀서가 생성한 레코드가 브로커로 전송되면 오프셋과 타임스탬프가 지정되어 저장된다.
* **브로커에 한번 적재된 레코드는 수정할 수 없고 로그 리텐션 기간 또는 용량에 따라서만 삭제된다.**
* 타임스탬프는 프로듀서에서 해당 레코드가 생성된 시점(createTime)의 유닉스 타입이 설정된다.
* 컨슈머는 레코드의 타임스탬프를 토대로 레코드가 언제 생성되었는지 알 수 있다.
* 다만, 프로듀서가 레코드를 생성할 때 임의의 타임스탬프 값을 설정할 수 있고, 토픽 설정에 따라 브로커에 적재된 시간으로 설정될 수 있다는 점을 유의해야 한다.
* 메시지 키는 메시지 값을 순서대로 처리하거나 메시지 값의 종류를 나타내기 위해 사용한다.
* 메시지 키를 사용하면 프로듀서가 토픽에 레코드를 전송할 때 메시지 키의 해시값을 토대로 파티션을 지정하게 된다. 
* 즉, 동일한 메시지 키라면 동일 파티션에 들어가는 것이다. 다만, 어느 파티션에 지정될지 알 수 없고 파티션 개수가 변경되면 메시지 키와 파티션 매칭이 달라지게 되므로 주의해야 한다.
* 만약 메시지 키를 사용하지 않는다면 프로듀서에서 레코드를 전송할 때 메시지 키를 선언하지 않으면 된다.
* 메시지 키를 선언하지 않으면 null로 설정된다.
* 메시지 키가 null로 설정된 레코드는 프로듀서 기본 설정 파티셔너에 따라서 파티션에 분배되어 적재된다.
* 메시지 값에는 실질적으로 처리할 데이터가 들어있다.
* 메시지 키와 메시지 값은 직렬화되어 브로커로 전송되기 떄문에 컨슈머가 이용할 때는 직렬화한 형태와 동일한 형태로 역직렬화를 수행해야 한다.
* 직렬화, 역직렬화할 때는 반드시 동일한 형태로 처리해야 한다. 
* 만약 프로듀서가 StringSerializer로 직렬화한 메시지 값을 컨슈머가 IntegerDeserializer로 역직렬화하면 정상적인 데이터를 얻을 수 없다.
* 레코드의 오프헷은 0 이상의 숫자로 이루어져 있다.
* 레코드의 오프셋은 직접 지정할 수 없고 브로커에 저장될 떄 이전에 전송된 레코드의 오프셋+1 값으로 생성된다.
* 오프셋은 카프카 컨슈머가 데이터를 가져갈 떄 사용된다.
* 오프셋을 사용하면 컨슈머 그룹으로 이루어진 카프카 컨슈머들이 파티션의 데이터를 어디까지 가져갔는지 명확히 지정할 수 있다.
* 헤더는 레코드의 추가적인 정보를 담는 메타데이터 저장소 용도로 사용한다.
* 헤더는 키/값 형태로 데이터를 추가하여 레코드의 속성(스키마 버전 등)을 저장하여 컨슈머에서 참조할 수 있다.
## 3.4 카프카 클라이언트
* 카프카 클러스터에 명령을 내려거나 데이터를 송수신하기 위해 카프카 클라이언트 라이브러리는 카프카 프로듀서, 컨슈머, 어드민 클라이언트를 제공하는 카프카 클라이언트를 사용하여 애플리케이션을 개발한다.
* 카프카 클라이언트는 라이브러리이기 떄문에 자체 라이플사이클을 가진 프레임워크나 애플리케이션 위에서 구현하고 실행해야 한다.
* ## 3.4.1 프로듀서 API
    * 카프카에서 데이터의 시작점은 프로듀서이다
    * 프로듀서 애플리케이션은 카프카에 필요한 데이터를 선언하고 브로커의 특정 토픽의 파티션에 전송한다.
    * 프로듀서는 데이터를 전송할 때 리더 파티션을 가지고 있는 ㅏ프카 브로커와 직접 통신한다.
    * 프로듀서를 구현하는 가장 기초적인 방법은 카프카 클라이언트를 라이브러리로 추가하여 자바 기본 애플리케이션을 만드는 것이다.
    * 프로듀서는 데이터를 직렬화하여 카프카 브로커로 보내기 때문에 자바에서 선언 가능한 모든 형태를 브로커로 전송할 수 있다.
    * 직렬화란 자바 또는 외부 시스템에서 사용가능하도록 바이트 형태로 데이터를 변환하는 기술이다.
    * 직렬화를 사용하면  프로듀서는 자바 기본형과 참조형뿐만 아니라 동영상, 이미지 같은 바이너리 데이터도 프로듀서를 통해 전송할 수 있다.
    * ## 카프카 프로듀서 프로젝트 생성
        * P.77
    * ## 프로듀서 중요 개념
        * 프로듀서는 카프카 브로커로 데이터를 전송할 때 내부적으로 파티셔너, 배치 생성 단계를 거친다.
        * 전송하고자 하는 데이터는 ProducerRecord 클래스를 통해 인스턴스를 생성했지만 ProducerRecord 인스턴스를 생성 필수 파라미터인 토픽과 메시지 값만 설정하였다.
        * ProducerRecord 생성 시 추가 파라미터를 사용하여 오버로딩하여 ProducerRecord의 내부 변수를 선언할 수도 있다.
        * 파티션 번호를 직접 지정하거나 타임스탬프를 설정, 메시지 키를 설정할 수도 있다.
        * 레코드의 타임스탬프는 카프카 브로커에 저장될 때 브로커 시간을 기준으로 설정되지만 필요에 따라 레코드 생성 시간 또는 그 이전/이후로 설정할 수도 있다.
        * KafkaProducer 인스턴스가 send() 메서드를 호출하면 ProducerRecord는 파티셔너에서 토픽의 어느 파티션으로 전송될 것인지 정해진다.
        * KafkaProducer 인스턴스를 생성할 때 파티셔너를 따로 설정하지 않으면 기본값인 DefaultPartitioner로 설정되어 파티션이 정해진다.
        * 파티셔너에 의해 구분된 레코드는 데이터를 전송하기 전에 어큐뮬레이터에 데이터를 버퍼로 쌓아놓고 발송한다.
        * 버퍼로 쌓인 데이터는 배치로 묶어서 전송함으로써 카프카의 프로듀서 처리량을 향상시키는 데에 상당한 도움을 준다
        * ## 파티셔너
            * 프로듀서 API를 사용하면 `UniformStickyPartitioner`와 `RoundRobinPartitioner` 2개 파티션을 제공한다
            * 카프카 클라이언트 라이브러리 2.5.0 버전에서 파티셔너를 지정하지 않은 경우 UniformStickyPartitioner가 파티셔너로 기본 설정된다.
            * UniformStickyPartitioner와 RoundRobinPartitioner는 둘 다 메시지 키가 이쓸 때는 메시지 키의 해시값과 파티션을 매칭하여 데이터를 전송한다는 점이 동일하다.
            * 메시지 키가 없을 때는 파티션에 최대한 동일하게 분배하는 로직이 들어있는데 UniformStickyPartitioner는 RoundRobinPartitioner의 단점을 개선하였다는 점이 다르다.
            * UniformStickyPartitioner는 프로듀서 동작에 특화되어 높은 처리량과 낮은 리소스 사용률을 가지는 특징이 있다.
            * 카프카 2.4.0 이전에는 UniformStickyPartitioner가 아닌 RoundRobinPartitioner가 기본 파티셔너로 설정되어 있었다.
            * RoundRobinPartitioner는 ProducerRecord가 들어오는 대로 파티션을 순회하면서 전송하기 때문에 배치로 묶이는 빈도가 적다.
            * 될 수 있으면 많은 데이터가 배치로 묶여 전송되어야 성능 향상을 기대할 수 있으므로 이를 해결하고자 카프카 2.4.0부터는 UniformStickyPartitioner가 기본 파티셔너로 설정 되었다.
            * UniformStickyPartitioner는 어큐뮬레이터에서 데이터가 배치로 모두 묶일 때까지 기다렸다가 배치로 묶인 데이터는 모두 동일한 파티션에 전송함으로써 RoundRobinPartitioner에 비해 향상된 성능을 가지게 되었다.
            * 카프카 클라이언트 라이브러리에서는 사용자 지정 파티셔너를 새엇ㅇ하기 위한 Partitioner 인터페이스를 제공한다
            * Partitioner 인터페이스를 상속받은 사용자 정의 클래스에서 메시지 키 또는 메시지 값에 따른 파티션 지정 로직을 적용할 수도 있다.
            * 파티셔너를 통해 파티션이 지정된 데이터는 어큐뮬레이터에 버퍼로 쌓인다
            * 샌더(sender) 스레드는 어큐뮬레이터에 쌓인 배치 데이터를 가져가 카프카 브로커로 전송한다
        * ## 압축
            * 추가적으로 카프카 프로듀서는 압축 옵션을 통해 브로커로 전송 시 압축 방식을 정할 수 있다.
            * 압축 옵션을 정하지 않으면 압축이 되지 않은 상태로 전송된다.
            * 카프카 프로듀서에서는 압축 옵션으로 gzip, snappy, lz4, zstd를 지원한다.
            * 압축을 하면 데이터 전송 시 네트워크 처리량에 이득을 볼 수 있지만 압축을 하는 데에 CPU 또는 메모리 리소스를 사용하므로 사용환경에 따라 적절한 압축 옵션을 사용하는 것이 중요하다.
            * 또한 프로듀서에서 압축한 메시지는 컨슈머 애플리케이션이 압축을 풀게 되는데 이때도 컨슈머 애플리케이션 리소스가 사용되는 점을 주의하자
    * ## 프로듀서 주요 옵션
        * 프로듀서 애플리케이션을 실행할 떄 설정해야 하는 필수 옵션과 선택 옵션이 있다.
        * ## 필수 옵션
            * bootstrap.servers
                * 프로듀서가 데이터를 전송할 대상 카프카 클러스터에 속한 브로커의 호스트 이름:포트를 1개 이상 작성한다.
                * 2개 이상 브로커 정보를 입력하여 일부 브로커에 이슈가 발생하더라도 접속하는 데에 이슈가 없도록 설정 가능하다.
            * key.serializer
                * 레코드의 메시지 키를 직렬화하는 클래스를 지정한다.
            * value.serializer
                * 레코드의 메시지 값을 직렬화하는 클래스를 지정한다.
        * ## 선택 옵션
            * acks
                * 프로듀서가 전송한 데이터가 브로커들에 정상적으로 저장되었는지 전송 성공 여부를 확인하는 데에 사용하는 옵션
                * `0`, `1`, `-1(all)` 중 하나로 설정할 수 있다. 설정값에 따라 데이터의 유실 가능성이 달라진다.
                * 기본값은 1로써 리더 파티션에 데이터가 저장되면 전송 성공으로 판단한다.
                * 0은 프로듀서가 전송한 즉시 브로커에 데이터 저장 여부와 상관없이 성공으로 판단한다.
                * -1 또는 all은 토픽의 `min.insync.replicas` 개수에 해당하는 리더 파티션과 팔로워 파티션에 데이터가 저장되면 성공하는 것으로 판단한다.
            * buffer.memory
                * 브로커로 전송할 데이터를 배치로 모으기 위해 설정할 버퍼 메모리양을 지정한다.
                * 기본값은 33554432(32MB)이다
            * retries
                * 프로듀서가 브로커로부터 에러를 받고 난 뒤 재전송을 시도하는 횟수를 지정한다.
                * 기본값은 2147483647이다.
            * batch.size
                * 배치로 전송할 레코드 최대 용량을 지정한다.
                * 너무 작게 설정하면 프로듀서가 브로커로 더 자주 보내기 때문에 네트워크 부담이 있고 너무 크게 설정하면 메모리를 더 많이 사용하게 되는 점을 주의해야 한다.
                * 기본값은 16384이다
            * linger.ms
                * 배치를 전송하기 전까지 기다리는 최소 시간이다
                * 기본값은 0이다
            * partitioner.class
                *레코드를 파티션에 전송할 때 적용하는 파티셔너 클래스를 지정한다.
                * 기본값은 org.apache.kafka.clients.producer.internals.DefaultPartitioner이다.
            * enable.idempotence
                * 멱등성 프로듀서로 동작할지 여부를 설정한다.
                * 기본값은 false이다
            * transactional.id
                * 프로듀서가 레코드를 전송할 때 레코드를 트랜잭션 단위로 묶을지 여부를 설정한다.
                * 프로듀서의 고유한 트랜잭션 아이디를 설정할 수 있다.
                * 이 값을 설정하면 트랜잭션 프로듀서로 동작한다.
                * 기본값은 null이다
    * ## 메시지 키를 가진 데이터를 전송하는 프로듀서
        * 메시지 키가 포함된 레코드를 전송하고 싶다면 ProducerRecord 생성 시 파라미터로 추가
        * 토픽이름, 메시지 키, 메시지 값을 순서대로 파라미터로 넣고 생성했을 경우 메시지 키가 지정
            ```java
            ProducerRecord<String, String> record = new ProducerRecord<>("test", "pangyo", "23);
            ```
        * 메시지 키와 메시지 값 확인
            ```bash
            bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic test --property print.key=true --property key.separator="-" --from-beginning
            ```
        * 파티션을 직접 지정하고 싶다면 토픽이름, 파티션번호, 메시지 키, 메시지 값을 순서대로 파라미터로 넣고 생성
        * 파티션 번호는 토픽에 존재하는 파티션 번호로 설정
            ```java
            int partitionNo = 0;
            ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, partitionNo, messageKey, messageValue);
            ```
    * ## 커스텀 파티셔너를 가지는 프로듀서
        * Partitioner 인터페이스를 사용하여 사용자 정의 파티셔너를 생성할 수 있다.
        * P.91
            ```java
            Properties configs = new Properties();
            configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS);
            configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
            configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
            configs.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, CustomPartitioner.class);
            
            KafkaProducer<String, String> producer = new KafkaProducer<>(configs);
            ```
        * 커스텀 파티셔너를 지정한 경우 ProducerConfig의 PARTITIONER_CLASS_CONFIG 옵션을 사용자 생성 파티셔너로 설정하여 KafkaProducer 인스턴스를 생성해야한다.
    * ## 브로커 정상 전송 여부를 확인하는 프로듀서
        * KafkaProducer의 send() 메서드는 Future 객체를 반환한다.
        * 이 객체는 RecordMetadata의 비동기 결과를 표현하는 것으로 ProducerRecord가 카프카 브로커에 정상적으로 적재되었느지에 대한 데이터가 포함되어있다.
        * send()의 결괏값은 카프카 브로커로부터 응답을 기다렸다가 브로커로부터 응답이 오면 RecordMetadata 인스턴스를 반환한다.
        * ```java
            ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, massgeValue);
            RecordMetadata metadata = producer.send(record).get();
            logger.info(metadata.toString());
            ```
        * 비동기로 결과를 확인할 수 있도록 Callback 인터페이스 사용
        * Callback 인터페이스를 사용하여 ProducerCallback 생성
        * ```java
            public class ProducerCallback implements Callback{
                private final static Logger logger = LoggerFactory.getLogger(ProducerCallback.class);

                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e){
                    if (e != null) {
                        logger.error(e.getMessage(), e);
                    } else {
                        logger.info(recordMetadata.toString());
                    }
                }
            }
            ```
        * ```java
            KafkaProducer<String, String> producer = new KafkaProducer<>(configs);
            ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC_NAME, messageValue);
            producer.send(record, new ProducerCallback());
            ```
* ## 3.4.2 컨슈머 API
    * ## 카프카 컨슈머 프로젝트 생성
    * ## 컨슈머 중요 개념
    * ## 컨슈머 주요 옵션
    * ## 동기 오프셋 커밋
    * ## 비동기 오프셋 커밋
    * ## 리밸런스 리스너를 가진 컨슈머
    * ## 파티션 할당 컨슈머
    * ## 컨슈머에 할당된 파티션 확인 방법
    * ## 컨슈머의 안전한 종료
* ## 3.4.1 어드민 API
    * ## 브로커 정보 조회
    * ## 토픽 정보 조회
## 3.5 카프카 스트림즈
* ## 3.5.1 스트림즈DSL
    * ## KStream
    * ## KTable
    * ## GlobalKTable
    * ## 스트림즈DSL 주요 옵션
    * ## 스트림즈DSL - stream(), to()
    * ## 스트림즈DSL - filter()
    * ## 스트림즈DSL - KTable과 KStream을 join()
    * ## 스트림즈DSL - GlobalKTable과 KStream을 join()
* ## 3.5.2 프로세서 API
## 3.6 카프카 커넥트
* ## 커넥트를 실행하는 방법
* ## 단일모드 커넥트
* ## 분산모드 커넥트
* ## 3.6.1 소스 커넥터
    * 파일 소스 커넥터 구현
## 3.7 카프카 미러메이커2
## 3.8 정리