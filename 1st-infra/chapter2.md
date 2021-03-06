### chapter2. 대규모 데이터 처리 입문 <sub>메모리와 디스크, 웹 애플리케이션과 부하</sub>

- 하테나의 데이터 규모를 예로 들어, DB의 index를 사용하지 않고 쿼리를 던질 때, 그리고 검색을 위해 쿼리를 던질 때, 200초 가까이 소모된다. 이는 사용자 입장에서 매우 답답함을 느낄 수 있는 시간이며, DB가 부하를 체감하고 있다는 결과를 도출할 수 있다. 데이터가 많으면 처리하는데 시간이 오래 걸린다는 것은, 직감으로는 알 수 있으나 정확히는 알 수 없다. 과연 어떤 원인 때문에 시간이 오래 걸리는지 조금 더 자세히 알아보도록 하자.

> 크고 작은 많은 문제에 대핮 노하우를 다수 보유하고 있는 것은, 뒤집어 말하면 그만큼의 트러블을 접해왔다는 것이다. 트러블이 발생하지 않았다면 애초에 그런 문제가 시스템에 내재하고 있다는 것을 좀체 알 수 없었을 거싱고, 데이터가 많아지면 '왠지 수고스러울 것 같아' 정도의 불안감은 있어도 실제로 어떤 노고가 있는지는 해보기 전까지는 몰랐을 것이다.
>
> (… 중략)
>
> '이렇게 되면 이렇게 해야 해!' 라는 원리를 누군가가 알고 있는 것이 아니라 시행착오의 연속으로 노하우가 축적되어 지금에 이르렀다는 느낌이다. 혹시 어딘가에 문제 해결 원리가 있지 않을까라고 생각하는 사람도 적지 않을 것이다. 그러나 실제로 발생하는 문제는 미지의 문제도 많은 것이 현실이다. **기본적인 사항은 파악해두고 '문제가 발생하면 그 자리에서 생각하자'라는 단순 명쾌한 생각도 필요하다는 것도 솔직한 생각이다.**



위의 인용문을 통해, 개발자는 단계에 있어서 어떠한 문제라도 직면할 수 있다는 것을 알 수 있다. 그리고 마지막 문장을 통해 직면한 문제에 대해 해결하고자 하는 의지는, 충분히 서비스를 안정적으로 운용할 수 있는 단계에 이르렀을 때 판단할 수 있을 것이라 생각된다. 만일 과정에 대한 정확한 이해와 원론적인 개념이 뒷받침되지 않는다면 **'그 자리에서 생각하자'** 라는 생각을 통해서도 절대 원인을 파악하고 답을 찾을 수 없을 것이라 생각된다. 



#### 대규모 데이터 처리의 어려운 점<sub>메모리와 디스크</sub>

- 대규모 데이터의 어려운 점을 단적으로 말하면, **'메모리 내에서 계산할 수 없다'** 는 것이다. 하지만 대규모의 데이터가 된다면, (통상적으로 4GB~ 16GB의 용량을 가진)메모리에서 감당할 수 없고, 결국 디스크를 두고 특정 데이터를 검색하게 된다. 그리고 디스크는 메모리에 비해 상당히 느리다. 

##### 메모리와 디스크의 속도 차

- 통상적으로 메모리와 디스크의 속도 차는 **10<sup>5</sup>~10<sup>6</sup>** 정도 난다. 이런 수치 감각은 꽤 중요하다.

##### 디스크는 왜 느린가?

-  메모리는 전기적인 부품이므로 물리적 구조는 탐색속도와 그다지 관계 없다. 하지만 디스크의 경우 아래의 그림과 같이 이루어져 있으므로 물리적 구조에 큰 영향을 받게 된다.

  ![디스크 구조](http://library.gabia.com/wp-content/uploads/2016/03/%EB%94%94%EC%8A%A4%ED%81%AC.png)

  디스크는 위의 그림과 같이 헤드 아래에 원반에 쌓여있다. 원하는 데이터를 찾으려고 할 때, 회전 중인 원반에서 헤드를 통해 위치를 찾고, 여러 겹으로 쌓여 있는 원반에서 다시 위치를 찾아야 한다. 이처럼 물리적인 동작을 수반하기 때문에 탐색 속도에 영향을 주게 된다. 하지만 만약 데이터가 모두 메모리에 올라와 있는 상태라면 탐색할 때, 물리적인 동작 없이 실제 데이터 탐색 시의 오버헤드가 거의 없으므로 빠르게 되는 것이다. 

##### OS 레벨에서의 연구 

- 디스크는 느리지만 OS의 동작을 통해 어느정도 커버하고 있다. 이는 chapter3에서 자세히 다룰 것이다.

##### 또 다른 요인 - 전송속도, 버스의 속도차

- CPU와 메모리 간에 연결되너 있는 버스는 7.5GB/초 정도로 굉장히 빠른 전송속도를 가진다. 하지만 디스크는 58MB/초 정도 밖에 되지 않는다. 이러한 차이 때문에 메모리에서 아무리 빨리 데이터를 읽을 수 있다고 해도, 디스크에서 읽어들이는 시간을 메모리는 기다릴 수 밖에 없다. 

> 이와 같이 현대의 컴퓨터에서는 메모리와 디스크 속도차를 생각하고 애플리케이션을 만들어야 한다. 이는 확장성을 생각할 때 매우 본질적이면서도 어려운 부분이다. 



#### 규모조정, 확장성 

- 앞서, 메모리와 디스크의 속도차를 통해 발생하는 부하에 대해 기본적으로 알아보았다. 그렇다면 메모리의 용량을 늘리는 것과 메모리의 개수를 늘리는 것과 같은 방법을 통해 해결할 수 있지 않을까? 전자의 경우 Scale up, 후자의 경우 Scale-out 전략이라 통칭하는데, 과연 효과적일까?

  - Scale up : 고가의 빠른 하드웨어를 사서 성능을 높이는 것
  - Scale out : 저가이면서 일반적인 성능의 하드웨어를 많이 나열해서 시스템 전체 성능을 올리는 것

  상황에 따라 다르다. 하드웨어 가격이 성능과 비례하지 않고, 다양한 패턴에 따라 달라진다. 어떤 경우는 Scale-up 방식이 효과적일 수 있고, 또 다른 상황에서는 Scale-out 방식이 적절할 수 있다. 또한 부하가 CPU에 걸리는지, I/O에 걸리는지에 따라서도 갈린다. 

- CPU 부하의 경우 Scale-out 전략이 효과적이다.

  - Http 요청을 받아 DB에 질의하고 응답받은 데이터를 다시 HTML로 클라이언트에 반환할 때는 기본적으로 CPU 부하만 소요된다. 그리고 이는 서버가 동일하게 작업을 처리하기만 하면 되므로 분산이 간단하다. 결국 새로운 서버를 추가하고자 한다면 원래 있던 서버와 완전히 동일한 구성을 갖는 서버를 추가하면 된다. (이 때 발생하는 어려움은 추후 설명한다.)
  - 같은 구성의 서버를 늘리고 로드밸런서로 분산한다ㅣ

- I/O 부하의 경우 Scale-out 전략이 효과적이지 않을 수 있다. (이는 효과적일 수도 있다는 의미를 내포한다.)

  - 대규모 환경에서 I/O 부하를 부담하는 서버는 애초에 분산시키기 어렵고, 디스크 I/O가 많이 발생하면 서버는 금새 느려진다. 또한 분산을 하더라도, 데이터는 어떠한 경로로 접촉하든 정확해야 한다. 여러 서버가 DB에 접촉함으로써 생기는 sync 맞추는 작업은 크게 신경써야 하는 부분이다. 



#### 두 종류의 부하와 웹 애플리케이션

- 일반적으로 부하는 크게 두 가지로 분류된다.

  - CPU 부하
  - I/O 부하

  CPU의 계산에 의존하는 프로그램(AP Server)의 경우 **CPU 바운드한 프로그램** 이라 칭하고 I/O를 통해 디스크에 잦은 접촉이 이뤄지는 서비스(DB Server)는 **I/O 바운드한 프로그램** 이라 칭한다. 



#### 프로그래머를 위한 대규모 데이터 기초

- 두가지 관점에서 설명한다.

  - 프로그램을 작성할 때의 요령
    1. 어떻게 하면 메모리에서 처리를 마칠 수 있을 까? (메모리는 속도가 빠르므로)
    2. 데이터량 증가에 강한 알고리즘을 사용하는 것
    3. 데이터 압출이나 검색기술과 같은 테크닉을 활용하는 것
  - 프로그램 개발의 근간이 되는 기초라는 점에서 전제로서 알아두었으면 하는 것
    1. OS 캐시
    2. 분산을 고려한 RDBMS 운용법
    3. 대규모 환경에서 알고리즘과 데이터 구조를 사용한다는 것은 어떤 것인가?

  



#### 그렇다면 부하가 생기는 위치를 개발자는 어떻게 찾을 수 있을까?

- 가장 중요한 생각은 **추측하지 말라! 계측하라!!!** 이다. 

  이는 서버 리소스의 이용현황을 정확하게 파악하고 부하가 걸리는 위치를 정확히 파악하라는 말이다. 

- 병목 규명작업의 기본적인 흐름

  1. Load Average 확인

     Load Average : CPU의 실행권한이 부여되기를 기다리는 프로세스의 수 + 디스크 I/O가 완료하기를 기다리고 있는 프로세스 / 단위시간

     - 궁금점 : 프로세스의 State중 Ready 상태인 프로세스와 Waiting 상태의 프로세스를 의미하는가?

  2. Load Average를 확인했다면 결국 부하가 걸리는 상태를 알 수 있다. 이 후 그 값이 CPU 부하인지, I/O 부하인지 확인하는 것

     - Linux에서는 sar 명령어를 통해 확인할 수 있다.

       ~~~ bash
       % sar
       00:00:01		cpu		%user		%nice		%system		%iowait		%steal		%idle
       00:00:01		all		59.84		0.00		1.54			0.00			0.00			38.62
       ...
       ...
       
       ~~~

       %user : 사용자 모드에서의 CPU 사용률

       %system : 시스템 모드에서의 CPU 사용률

       %iowait : I/O 대기율

       - Load Average가 높고 %user와 %system의 비율이 높다면 CPU에서의 부하
       - Load Average가 높고 %iowait의 비율이 높다면 I/O에서의 부하

       