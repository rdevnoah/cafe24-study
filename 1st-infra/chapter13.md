## Chapter13 다중성 확보, 시스템 안정화 <sub>100%에 근접한 가동률을 실현하는 원리</sub>

서버를 분산하여 다중성을 확보하는데에 가장 중요한 개념은 SPOF(Single Poing of Failure)를 극복하는 것이다. 웹 서비스의 경우 365일 100%의 근접한 가동률을 목표로 한다. 만일, 서버 한대로만 운영중인 서비스가 있다고 가정하면, 그 한대에 문제가 생기면 모든 서비스가 마비된다. 이것이 SPOF이고, 이것을 극복하는 것이 과제이다. 

#### 다중성 확보 - AP 서버

- AP 서버에 대한 대응으로는 로드밸런서로 failover -> failback을 통해 문제가 발생한 서버를 자동으로 분리하고, 서버가 복구되면 원래 상태로 복귀시키는 작업을 수행한다.
  - failover : 문제가 생긴 서버를 자동으로 분리하는 것
  - failback : 정상이 되면 복귀시키는 처리
- 로드 밸런서는 서버에 대해 주기적으로 헬스체크가 이뤄지며 AP서버나 DB서버가 살아있는지 여부를 판정한다.

![로드밸런서](https://t1.daumcdn.net/cfile/tistory/2413C14E590B4F8A3A)

- 로드 밸런서를 통한 failover, failback 과정
  - 만일 위의 그림에서 오른쪽 서버 (1, 2, 3)와 같이 분산되어있다고 가정할 때, 1번 서버가 다운되면 로드밸런서가 다운된 것을 인식하여 그쪽으로의 접속을 차단한다. 그 후 2, 3 서버만으로 처리를 수행한다.
  - 1 서버의 상태가 복구되면 다시 복귀시켜 1, 2, 3 서버 모두 요청을 처리하도록 한다.



#### 다중성 확보 - DB서버

> DB 서버도 마찬가지로 서버를 여러 대 나열해서 1, 2대 정지하더라도 충분한 처리능력이 있도록 해두는 것이 중요하다. 또한 마스터의 다중화도 수행하고 있다.

- 하테나에서 사용하는 Master의 다중화 방법

  - 쌍방의 레플리케이션. 즉, 서로가 서로의 Slave가 되는 상태로 해두고, 한쪽에 쓰기작업을 하면 다른 한쪽으로 전달하고 반대쪽에 쓰더라도 다른 쪽으로 전달하는, 양방향 레플리케이션 방법이다.

    ![양방향 레플리케이션](https://t1.daumcdn.net/cfile/tistory/99713F355AA0C4EE1F) 

  - 위의 그림과 같은 방식이다. 서로가 서로의 Master이자 Slave이기 때문에 양쪽의 서버에 모두 갱신 쿼리가 이뤄질 수 있다. 

    - **하지만 주의해야 할 점은, 한 쪽으로 쓰기 요청이 이뤄졌을 때 반대쪽으로 전달되는 흐름이 추가되어 싱크를 맞춰야 하기 때문에, 밀리초 단위로 보면 데이터가 일치하지 않는 상태가 항상 존재한다.**



#### 멀티마스터의 failover, failback

1. 마스터 상호 간에 VRRP라는 프로토콜을 활용하여 서로 감시한다. 
2. 기본적으로 멀티 마스터는 Active/Standby 구성을 하고 있다. 한쪽은 Active이고 한 쪽은 Standby이다. 
   - Active인 서버가 분리되면 자동으로 Standby인 서버가 Active가 되며, 쿼리를 수행한다.

- VRRP (Virtual Router Redundancy Protocol)
  - 주로 사용되는 Active서버에 가상의 IP를 부여한다. 
  - 예를 들어 서버 두대가 각각 0.1, 0.2 의 IP를 가지고 있지만 요청을 수행하는 Active 서버에 0.3이라는 가상의 IP를 부여하여 사용자의 요청은 0.3으로의 접속만 수행하도록 한다. 결국 0.1이 고장나면 0.3의 프로토콜만 0.2로 옮겨주면 사용자는 0.3을 계속 사용해서 0.2로도 접속할 수 있는 것이다. 결국 여러 서버 IP를 하나의 IP로 접속 할 수 있도록 한다.



#### 다중성 확보 - 스토리지 서버

> 하테나에서는 분산 스토리지 서버로 MogileFS를 사용한다.

아래의 그림은 하테나에서 사용중인 MofileFS의 구성도이다.

![MogileFS](https://blog.sixapart.jp/images/mogilefs.png)

- 실제 파일은 storage nodes에 위치하며, 특정 URL에 대한 실제 파일의 위치를 나타내는 메타 정보를 DB로(tracker's database) 관리하는 심플한 형태이다. 
- 다양한 분산 파일시스템이 있으며 추후에 알아보고 동작원리를 대략적이나마 파악하는것도 큰 도움이 될 듯하다.



**참고 : 하테나의 경우 규모가 작을 때, NFS(Network File System) 마운트를 사용했는데 지금은 사용하지 않는 이유**

- NFS의 경우 네트워크로(TCP/IP, UDE etc..) 연결되어있다. 네트워크 상에서 발생하는 병목으로 인해 시스템이 아예 다운되어버리는 경우가 종종 발생한다.
- 네트워크를 사용함으로써 발생하는 병목을 제어하기 힘들어 대용량(TB단위)의 파일을 위한 스토리지 서버로 활용하기에는 만들어진 기원 자체가 맞지 않는다.



### 시스템 안정화

##### 시스템 안정화를 위한 trade-off(상반관계)가 존재한다.

- 안정성과 자원효율 간의 trade-off

- 안정성과 속도 간의 trade-off

  - 위의 두 가지 모두를 설명하기 위해 가정을 하자.

    - 빠듯할 정도로 메모리를 튜닝하여 90% 이상이 사용되도록 하였다면, 처리해야 할 데이터량이 갑자기 늘어나거나 애플리케이션에 버그가 있거나, 메모리 누수가 발생해서 갑자기 사용량이 치솟으면 곧바로 swap을 사용하여 성능이 저하되거나, 서비스 장애가 일어난다.

    - CPU도 마찬가지이다. 

      > 한계에 이를 때까지 메모리 튜닝
      >
      > - 메모리 소비가 늘어난다. -> 성능 저하 -> 장애
      >
      > 한계에 이를 때까지 CPU 사용
      >
      > - 서버 1대가 다운 -> 전체 처리능력 초과 -> 장애

- 실제로는, 위의 상황이 발생하지 않도록 메모리와 CPU의 사용율을 70%(혹은 60%, 50%, etc…) 정도로 유지하는 등 어느 정도 여유를 가지도록 설계하는 것이 중요하다.

#### 시스템의 불안정 요인

- 애플리케이션/서비스 레벨에서의 요인 -> 부하 증가

  1. 기능 추가

     - 기능 추가는 서비스가 예상보다 무거워져 전체적인 부하가 늘어난다.

  2. 메모리 누수

     - 하테나의 경우와 같이 Perl과 같은 Lightweight Language를 사용하면, 메모리 누수를 완전 배제하기 힘들다. 

       > 메모리 점유와 해제와 같은 세세한 컨트롤이 Lightweight Language에서는 힘들어서일까?

     - 메모리 누수 : 프로세스나 쓰레드의 데드락과 같이 자원을 반납하지 않고 계속 메모리를 점유하고 있는 것. 결국 메모리가 부족해져서 장애가 발생한다.

  3. 지뢰

     - 특정 URL이 읽히면 서버에 장애가 일어나는 것. 지뢰의 원인은 보통 찾기가 매우 힘들다. 기술자다운 테크닉이 필요한데, 디버거를 활용하여 찾는 노하우가 필요하다.

  4. 사용자의 액세스 패턴

     - 예시로, 인기 포탈 사이트에 우리의 서비스가 메인을 장식했다. -> 사용자의 요청이 갑자기 증가한다. -> 장애 발생
     - 전형적으로는 Squid와 같은 캐시 서버를 추가해서 통상적인 사용자가 아닌 게스트의 경우 본 서버로 가는 무서운 요청 처리보다는 캐시 서버를 통한 가벼운 처리를 하도록 한다.

  5. 데이터량 증가

     - 당연한 일이다.

  6. 외부연계 추가(API 추가)

     - 외부 API가 우리의 서비스와 상관없이 다운될 수도 있으므로, 덩달아 우리의 서비스가 다운될 수도 있다.

- 하드웨어 레벨에서의 요인 -> 처리능력 저하

  1. 메모리, HDD 장애
  2. NIC 장애



#### 시스템 안정화 대책

##### 1. 적절한 버퍼 유지

- 시스템 수용 능력의 70% 정도로 여분의 버퍼를 마련하는 것. 이를 넘어서면 서버를 추가하는 것과 같은 추가 작업이 이뤄진다.

#####2. 불안정 요인 제거

- SQL 부하대책
  - 엔지니어는 부하가 높아질 만한 SQL을 가능한 한 파악해두고, 그 쿼리가 발생하면 해당 용도를 위해 격리한 DB로 보낸다.
- 메모리 누수 줄이기
  - 지속적인 감시가 필요하다.
- 비정상 동작 시 자율 제어
  1. 자동 DoS 판정
     - 동일한 IP 주소로부터 짧은 시간대에 잦은 요청이 오면 자동으로 DoS 판정을 내려 요청의 부하를 막는 행위를 할 수 있다.
  2. 자동 재시작
     - 메모리 누수로 인해 swap을 사용하면 느려진다. -> 일정 기간을 기준으로 각 서버를 자동으로 재시작 할 수 있는 시스템 마련
     - *chapter14에서 다룰 가상화 기술을 확인하면 어떻게 서버를 원격에서 재시작할 수 있는지 알 수 있다.*
  3. 자동 쿼리제거
     - 요청이 오래 걸리는 쿼리를 일정 시간이 지나도 결과값이 돌아오지 않는다면 자동으로 KILL 하는 것.
