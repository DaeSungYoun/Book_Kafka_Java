# 2. 카프카 빠르게 시작해보기
## 2.1 실습용 카프카 브로커 설치
* ## 2.1.1 AWS EC2 인스턴스 발급 및 보안 설정
    * AWS에서 카프카를 구축할 수 있는 방법은 두 가지가 있다.
    * 첫 번째는 MSK(Managed Service Kafka)를 사용, 두 번째는 EC2에서 인스턴스를 발급받아서 설치 및 실행
    * MSK는 AWS에서 공식으로 지원하는 완전 관리형 카프카 서비스이다
* ## 2.1.2 인스턴스에 접속하기
    * ssh 명령어로 접속하기
        * (내용 정리 예정)
    * putty로 접속하기
        * (내용 정리 예정)
* ## 2.1.3 인스턴스에 자바 설치
    * 카프카 브로커를 실행하기 위해서는 JDK 필요
    * 카프카 브로커는 스칼라와 자바로 작성되어 JVM 환경 위에서 실행됨
* ## 2.1.4 주키퍼, 카프카 브로커 실행
    * ### 카프카 브로커 힙 메모리 설정
        * 카프카 브로커는 레코드의 내용은 페이지 캐시로 시스템 메모리를 사용하고
        * 나머지 객체들을 힙 메모리에 저장하여 사용한다는 특징이 있다.
        * **이러한 특징으로 카프카 브로커를 운영할 때 힙 메모리를 5GB 이상으로 설정하지 않는 것이 일반적이다**
        * 카프카 브로커 실행 시 힙 메모리를 지정하고 싶다면 KAFKA_HEAP_OPTS 환경변수를 힙 메모리 사이즈와 함께 지정
        ```
        export KAFKA_HELP_OPTS="-Xmx400m -Xms400m"
        echo $KAFKA_HEAP_OPTS (입력)
        -Xmx400m -Xms400m (출력)
        ```
        * 터미널에서 사용자가 입력한 KAFKA_HEAP_OPTS 환경변수는 터미널 세션이 종료되고 나면 다시 초기화되어 재사용이 불가하다.
        * 이를 해결하기 위해 KAFKA_HEAP_OPTS 환경변수 선언문을 ~/.bashrc 파일에 넣으면 된다.
        * **카프카 브로커 실행 시 메모리를 설정하는 부분은 카프카를 실행하기 위해서 사용하는 kafka-server-start.sh 스크립트 내부에서 확인 가능**
    * ### 카프카 브로커 실행 옵션 설정
        * config 폴더에 있는 server.properties 파일에는 카프카 브로커가 클러스터 운영에 필요한 옵션들을 지정할 수 있다.
        * 이미 실행되고 있는 카프카 브로커의 설정을 변경하고 싶다면 브로커를 재시작해야 하므로 신중히 설정 필요
        * config/server.properties
            * (내용 정리 예정, P.35)
    * ### 주키퍼 실행
        * 분산 코디네이션 서비스를 제공하는 주키퍼는 카프카의 클러스터 설정 리더 정보, 컨트롤러 정보를 담고 있어 카프카를 실행하는 데에 필요한 필수 애플리케이션이다.
        * 주키퍼를 상용환경에서 안전하게 운영하기 위해서는 3대 이상의 서버로 구성하여 사용
        * -daemon옵션과 주키퍼 설정 경로인 config/zookeeper.properties와 함께 주키퍼 시작 스크립트인 bin/zookeeper-server-start.sh를 실행하면 주키퍼를 백그라운드에서 실행할 수 있다
        * 만약 주키퍼 시작 스크립트를 넣었는데도 불구하고 정상적으로 실행되지 않는다면 -daemon 옵션을 제외하고 실행해 보자. 실행과 함께 주키퍼 로그를 확인할 수 있으므로 무엇이 문제인지 확인할 수 있다.
            ```
            bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
            ```
        * 주키퍼가 정상적으로 실행되었는지 jps 명령어로 확인할 수 있다. jps는 JVM 프로세스 상태를 보는 도구로서 JVM 위에서 동작하는 주키퍼의 프로세스를 확인할 수 있다
        * -m 옵션과 함께 사용하면 main 메서드에 전달된 인자를 확인할 수 있고, -v 옵션을 사용하면 JVM에 전달된 인자(힙 메모리 설정, log4j 설정 등)를 함께 확인 할 수 있다.
            ```
            jps -vm
            ```
    * ### 카프카 브로커 실행 및 로그 확인
        * -daemon 옵션으로 백그라운드 모드로 실행 가능
            ```
            bin/kafka-server-start.sh -daemon config/server.properties
            ```
            ```
            jps -m
            ```
            ```
            tail -f logs/server.log
            ```
* ## 2.1.5 로컬 컴퓨터에서 카프카와 통신 확인
    * ### 로컬 컴퓨터에 카프카 설치
        ```
        1. curl https://archive.org/dist/kafka/2.5.0/kafka_2.12-2.5.0.tgz
        2. tar -xvf kafka.tgz
        3. cd kafka_2.12-2.5.0
        4. ls (목록 확인)
        5. ls bin (bin 폴더 목록 확인)
        6. bin/kafka-broker-api-versions.sh --bootstrap-server 13.124.99.218:9092(카프카의 버전, broker.id, rack 정보, 각종 카프카 브로커 옵션들 확인 가능)
        ```
        > 카프카 브로커와 로컬 커맨드 라인 툴 버전을 맞춰야 하는 이유<br><br>
        카프카 브로커로 커맨드 라인 툴 명령을 내릴 때 브로커의 버전과 커맨드 라인 툴 버전을 맞춰서 사용하는 것을 권장한다.<br>
        브로커의 버전이 업그레이드됨에따라 커맨드 라인 툴의 상세 옵션이 달라져서 버전 차이로 인해 명령이 정상적으로 실행되지 않을 수도 있기 때문이다.
    * ### 테스트 편의를 위한 hosts 설정
        * (P.44)
## 2.2 카프카 커맨드 라인 툴
* ## 2.2.1 kafka-topics.sh
    > 토픽을 생성하는 2가지 방법<br><br>
    1.카프카 컨슈머 또는 프로듀서가 카프카 브로커에 생성되지 않은 토픽에 대해 데이터를 요청할 때<br>
    2.커맨드 라인툴로 명시적으로 토픽을 생성<br>
    토픽을 효과적으로 유지보수하기 위해서는 토픽을 명시적으로 생성하는것을 추천한다. 토픽마다 처리되어야 하는 데이터의 특성이 다르기 때문이다.<br>
    토픽을 생성할 때는 데이터 특성에 따라 옵션을 다르게 설정할 수 있다.
    * ### 토픽 생성
        * 토픽 이름은 내부 데이터가 무엇이 있는지 유추가 가능할 정도로 자세히 적는 것을 추천
        ```
        bin/kafka-topics.sh --create --bootstrap-server my-kafka:9092 --topic hello.kafka
        ```
        ```
        bin/kafka-topics.sh --create --bootstrap-server my-kafka:9092 --partitions 3 --replication-factor 1 --config retention.ms=172800000 -topic hello.kafka.2
        ```
        * "--partitions 3"
            * 파티션 개수 지정
            * 파티션 최소 개수는 1개이다
            * 만약 이 옵션을 사용하지 않으면 카프카 브로커 설정파일(config/server.properties)에 있는 num.partitions 옵션값을 따라 생성됨
        * "--replication-factor 1"
            * 토픽의 파티션을 복제할 복제 개수를 적는다.
            * 1은 복제를 하지 않고 사용한다는 의미
            * 2이면 1개의 복제본을 사용하겠다는 의미
            * 파티션의 데이터는 각 브로커마다 저장된다. 한 개의 브로커에 장애가 발생하더라도 나머지 한 개 브로커에 저장된 데이터를 사용하여 안전하게 데이터 처리를 지속적으로 할 수 있다.
            * 복제 개수의 최소 설정은 1이고 최대 설정은 통신하는 카프카 클러스터의 브로커 개수이다.
            * 실제 업무환경에서는 3개 이상의 카프카 브로커로 운영하는 것이 일반적이고 2 또는 3으로 복제 개수를 설정하여 사용한다
            * 만약 이 설정을 명시적으로 하지 않으면 카프카 브로커 설정에 있는 default.replication.factor 옵션값에 따라서 생성된다
        * "--config retention.ms=172800000"
            * --config를 통해 kafka-topics.sh 명령에 포함되지 않은 추가적인 설정을 할 수 있다.
            * retention.ms는 토픽의 데이터를 유지하는 기간을 뜻한다.
            * 172800000ms == 2일
        > 토픽 생성 시 --zookeeper가 아니라 --bootstrap-server 옵션을 사용하는 이유<br><br>
        카프카 2.1을 포함한 이전 버전에서는 일부 카프카 커맨드 라인툴이 주키퍼와 직접 통신하여 명령을 실행 했다<br>
        그러나 카프카 2.2 버전 이후로는 주키퍼와 통신하는 대신 카프카를 통해 토픽과 관련된 명령을 실행할 수 있게 되었다.
    * ### 토픽 리스트 조회
        ```
        bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --list
        ```
        * 내부 관리를 위한 인터널 토픽(internal topic)이 존재하는데 조회 시 목록에서 제외하기
            ```
            bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --list --exclude -internal
            ```
    * ### 토픽 상세 조회
        * 이미 생성된 토픽의 상태를 --describe 옵션을 사용하여 확인할 수 있다.
        * 파티션 개수가 몇 개인지, 복제된 파티션이 위치한 브로커의 번호, 기타 토픽을 구성하는 설정들을 출력한다.
        * 토픽이 가진 파티션의 리더가 현재 어느 브로커에 존재하는지도 같이 확인할 수 있다.
            ```
            bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --describe --topic hello.kafka.2
            ```
    * ### 토픽 옵션 수정
        * 파티션 개수를 변경하려면 kafka-topics.sh 사용
            * --alter 옵션과 --partitions 옵션을 함께 사용하여 파티션 개수를 변경할 수 있다.
            * 토픽의 파티션은 늘릴 수 있지만 줄일 수는 없다. 그러므로 파티션 개수를 늘릴 때는 반드시 늘려야 하는 상황인지 판단을 하는 것이 중요하다.
                ```
                bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --alter --partitions 4
                ```
        * retention 기간 변경은 kafka-configs.sh 사용
            * retention.ms를 수정하기 위해 kafka-configs.sh와 --alter, --add-config 옵션을 사용
            * --add-config 옵션을 사용하면 이미 존재하는 설정값은 변경하고 존재하지 않는 설정값은 신규로 추가한다.
                ```
                bin/kafka-configs.sh --bootstrap-server my-kafka:9092 --entity-type topics --entity-name hello.kafka --alter --add-config retetion.ms=86400000
                ```
* ## 2.2.2 kafka-console-producer.sh
    * 레코드 전송
        ```
        bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --property "parse.key=true" --property "key.separator=:"
        ```
        * --property "parse.key=true"
            * parse.key를 true로 설정하면 레코드 전송할 때 메시지 키를 추가할 수 있다.
        * --property "key.separator=:"
            * 메시지 키와 메시지 값을 구분하는 구분자를 선언
            * key.separator를 선언하지 않으면 기본 설정은 Tab delimiter(\t)이다.
            * key.separator로 사용하는 구분자를 넣지 않고 메시지를 보내면 KafkaException과 함께 종료됨
    * 토픽에 넣는 데이터는 레코드라고 부르며 메시지 키와 메시지 값으로 이루어져 있다.
    * kafkaa-console-producer.sh로 전송되는 레코드 값은 UTF-8을 기반으로 Byte로 변환되고 ByteArraySerializer로만 직렬화 된다
    * 즉, String이 아닌 타입으로는 직렬화 하여 전송할 수 없다.
    * 텍스트 목적으로 문자열만 전송할 수 있고, 다른 타입으로 직렬화하여 데이터를 브로커로 전송하고 싶다면 카프카 프로듀서 애플리케이션을 직접 개발해야한다. 
    * 메시지 키가 null인 경우에는 프로듀서가 파티션으로 전송할 때 레코드 배치단위(레코드 전송 묶음)로 라운드 로빈으로 전송
    * 메시지 키가 존재하는 경우에는 키의 해시값을 작성하여 존재하는 파티션중 한 개에 할당된다.
        * 메시지 키가 동일한 경우에는 동일한 파티션으로 전송
    * 다만, 이런 메시지 키와 파티션 할당은 프로듀서에서 설정된 파티셔너에 의해 결정되는데, 기본 파티셔너의 경우 이와 같은 동작을 보장한다.
    * 커스텀 파티셔너를 사용할 경우에는 메시지 키에 따른 파티션 할당이 다르게 동작할 수도 있으니 참고
    > 파티션 개수가 늘어나면 새로 프로듀싱되는 레코드들은 어느 파티션으로 갈까?<br><br>
    메시지 키를 가진 레코드의 경우 파티션이 추가되면 파티션과 메시지 키의 일관성이 보장되지 않는다.<br>
    즉, 이전에 메시지 키가 파티션 0번에 들어갔었다면 파티션을 늘린 뒤에는 파티션 0번으로 간다는 보장이 없다.<br>
    만약 파티션을 추가하더라도 이전에 사용하던 메시지 키의 일관성을 보장하고 싶다면 커스텀 파티셔너를 만들어서 운영해야 한다.

* ## 2.2.3 kafka-console-consumer.sh
    * ### 토픽 데이터 가져오기
        ```
        bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --from-beginning
        ```
        * --from-beginning
            * 토픽에 저장된 가장 처음 데이터부터 출력
        ```
        bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --property print.key=true --property key.separator="-" group hello-group --from-beginnig
        ```
        * --property print.key=true
            * 메세지 키 확인, 기본설정값이 fasle이기 떄문에 true로 설정
        * --property key.separator="-"
            * 메시지 키 값을 구분하기 위해 설정
        * --group
            * 신규 컨슈머 그룹 생성. 컨슈머 그룹은 1개 이상의 컨슈머로 이루어져 있다.
            * 이 컨슈머 그룹을 통해 가져간 토픽의 메시지는 가져간 메시지에 대해 커밋을 한다.
            * 커밋이란 컨슈머가 특정 레코드까지 처리를 완료했다고 레코드의 오프셋 번호를 카프카 브로커에 저장하는 것.
            * 커밋 정보는 _consumer_offsets 이름의 내부 토픽에 저장
        * 순서
            * 토픽에 넣은 데이터의 순서를 보장하고 싶다면 가장 좋은 방법은 파티션 1개로 구성된 토픽을 만드는 것이다
            * 한 개의 파티션에서는 데이터의 순서를 보장하기 떄문이다.
* ## 2.2.4 kafka-console-groups.sh
    * ### 컨슈머 그룹 리스트 확인
        ```
        bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 --list
        ```
    * ### 컨슈머 그룹 상세 조회
        ```
        bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 --group hello-group --describe
        ```
        * --group {값}
            * --describe 옵션과 함께 사용되며 조회할 그룹명을 입력
        * --describe
            * 컨슈머 그룹의 상세 내용 확인
        * p.57
* ## 2.2.5 kafka-verifiable-producer, consumer.sh
    * kafka-verifiable로 시작하는 2개의 스크립트를 사용하면 String 타입 메시지 값을 코드 없이 주고 받을 수 있다.
        ```
        bin/kafka-verifiable-producer.sh --bootstrap-server my-kafka:9092 --max-messages 10 --topic verify-test
        ```
        * --bootstrap-server {값}
            * 통신하고자 하는 클러스터 호스트와 포트입력
        * --max-messages {값}
            * kafka-verifiable-producer.sh로 보내는 데이터 개수를 지정
            * 만약 -1을 옵션값으로 입력하면 kafka-verifiable-producer.sh가 종료될 떄까지 계속 데이터를 토픽으로 보낸다.
        * --topic {값}
            * 데이터를 받을 대상 토픽 입력
        ```
        bin/kafka-verifiable-consumer.sh --bootstrap-server my-kafka:9092 --topic verify-test --group-id test-group
        ```
        * --bootstrap-server {값}
            * 통신하고자 하는 클러스터 호스트와 포트입력
        * --topic {값}
            * 데이터를 가져오고자 하는 토픽 입력
        * --group-id
            * 컨슈머 그룹을 지정
* ## 2.2.4 kafka-delete-records.sh
    * 카프카에서는 토픽의 파티션에 저장된 특정 데이터만 삭제할 수는 없다.
    * 파티션에 존재하는 가장 오래된 오프셋부터 지정한 오프셋까지 삭제
        ```
        bin/kafka-delete-records.sh --bootstrap-server my-kafka:9092 --offset-json-file delete-topic.json
        ```