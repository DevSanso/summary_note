# Scylla 컨테이너 실행 및 기본 세팅법

### 목차
1. 컨테이너 매개변수
2. 컨테이너 실행
3. trace 

#### 컨테이너 매개변수
* smp <int>: cpu 코어중 어느정도를 사용할지에 대한 옵션, 다만 특정 cpu 코어를 사용하도록 세부 설정이 필요할 경우 cpuset 옵션 사용 
* seeds <ip>,<ip2>.... : 현재 설정 하고자 한  시드 노드들의 ip 정보를 입력 가능한 매개변수, 만약 입력이 없을 시 해당 컨테이너 자체가 시드 노드가 된다(scylladb는 클러스터에서 최초의 시드 노드가 무조건 있어야 하기 때문에 사용하는걸로 추측)
* listen-address <ip> : 서버의 bind 할 ip 설정 옵션
* memory <count> : 메모리 사용량 제한, 1M 혹은 1G 같이 설정하면 된다.
* broadcast-address <ip> | broadcast-rpc-address <ip>
 : 다른 ip의 클러스트하고 연결 및 통신 되도록 설정 하는 옵션
* reserve-memory <count> : 컨테이너 예약 메모리 사용량 제한 설정, memory 옵션하고 사용법은 
* overprovisioned <1 | 0> :  해당 옵션을 사용하면, 현재 필요 사용량보다 더 추가적인 리소스 할당 여부를 설정하는 옵션, 갑작스런 과부하 현상에서 빠른 대응이 되지만, 과다 할당으로 문제가 발생할수도 있음
* developer-mode  <1 | 0> : 개발 모드로 실행 여부 확인, scylladb가 개발 모드 안하실 리소스 설정에 따라서 실행이 안될 경우가 있음, 기본값은 활성화 이며, 필요시 끄는 옵션 추가가 필요
* authenticator <authenticator name> : db로 로그인할때 사용하고자 하는 authenticator 설정이며, 기본값은 allowallauthenticator이다. PasswordAuthenticator로 설정 하지 않으면, 모든 연결이 익명 계정으로 로그인 되면, 일부 테이블 조회 및 생성이 안되니, 최소 PasswordAuthenticator로 설정이후 사용 권고(공식 문서에는 기재 되어 있으나, 실행 시에는 에러 발생)   

* authorizer <authorizer name> : authorizer 설정, 기본값은 allowallauthorizer이며, 해당값을 사용하면, permissions에 문제가 발생할수 있으니, 운영하실에는 CassandraAuthorizer 사용을 권고, 해당 옵션의 은 system_auth.permissions 테이블에서 조회 가능


* alternator-address <ip> | alternator-port <port> | alternator-write-isolation <policy><br /> : aws의 DynamoDB API를 해당 노드에서 사용할수 있도록 만든 옵션으로, 기본값은 listen-address를 따라가며, aws 서비스 사용 안하실 신경 안쓰여도 될것 같다
다른 옵션들은 [_scylladb의 docker  컨테이너 페이지_](https://hub.docker.com/r/scylladb/scylla/)를 
#### 

#### 컨테이너 실행
단순 테스트를 사용하고자 하면, smp, memory, authenticator 그리고 developer-mode 옵션만 사용하면 춤분할것으로 보임
(포트 관련 내용은 [_해당 문서_](https://opensource.docs.scylladb.com/stable/operating-scylla/security/security-checklist.html)를 참고)

1. docker run --name scylladb --hostname scylladb -p 9042:9042 -d  scylladb/scylla --smp 1 --memory 1G --developer-mode 1
2. docker exec -it scylla bash로 root 계정 
3. apt update; apt install -y 
4. vim vim /etc/scylla/scylla.yaml 실행 하여서 authenticator 값 PasswordAuthenticator로 변경

그 이후 호스트환경에서 접근하고자 하면, pip install scylla-cqlsh를 설치하여서 사용
(기본 계정 : cassandra/cassandra)

### trace 기능 
nodetool settraceprobability 1 실행하면 trace 기능 활성화로 추측, [_해당문서_](https://opensource.docs.scylladb.com/stable/using-scylla/tracing.html)  참고
