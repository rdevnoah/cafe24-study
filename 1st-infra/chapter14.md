## chapter 14 효율향상전략

## 하드웨어의 리소스 사용률 높이기

### 가상화 기술

- 가상화 기술의 목적

  > 확장성
  >
  > - 오버헤드의 최소화
  >
  > 비용대비 성능
  >
  > - 리소스 사용률 향상
  > - 운용의 유연함(환경의 단순화)
  >
  > 고가용성
  >
  > - 환경의 격리

*참고:하테나에서 사용중인 가상화 기술에 대한 설명으로, 대략적인 흐름을 파악할 수 있다. 하테나에서 가상화 기술을 사용함으로써 얻을 수 있는 효용은 아래와 같다.*

1. IPMI를 대체하는 하이퍼바이저
2. 하드웨어간 차이 흡수(->환경 추상화)
3. 준 가상화(ParaVitualization) 사용
4. 리소스 소비 제어
   - 과부하 경고
   - 부하 조정

그럼 각 효용에 대해 살펴보도록 하자.

1. IPM를 대체하는 하이퍼바이저
   - 하이퍼바이저란, 이른바 호스트OS 이다. 호스트 컴퓨터 1대에서 다수의 운영체제를 동시에 실행할 수 있도록 해주는 가상 플랫폼 기술을 이르며, 하테나에서는 가상 서버 상에 최초로 기동하는 OS(*Dom0*)를 하이퍼바이저라고 부른다. 
     - Dom0 : Xen(CentOS)에서 서버에 최초로 올라가는 OS이며, 하테나에서는 하이퍼바이저라 부른다.
     - DomU : 호스트 OS 위에서 기동하는 게스트 OS를 말한다.
     - 간단히 말하면, 지금 작업하고 있는 맥북의 경우, Dom0는 OSX 이고, OSX 위에서 가상환경으로 돌아가는 CentOS는 DomU라고 할 수 있다.

2. 하드웨어간 차이 흡수
   - 하드웨어간의 차이에 상관없이 환경을 추상화 하기 때문에, 새로운 하드웨어나 오래된 하드웨어로도 차분에 신경쓰지 않고 사용할 수 있다.
   - ~~이 말은 오래된 하드웨어에서도 환경을 추상화, 새로운 하드웨어에서도 환경을 추상화 하기 때문에 결국 상관이 없다는 말일까?~~
   - 아니면, 여러 하드웨어를 같이 사용하여도 공통으로 환경을 추상화한다는 말일까?

3. 준 가상화 사용
   - Xen에 특화된 얘기이기는 하지만, 준 가상화(ParaVirtualization)의 개념에 대해서 학습한다.
   - 용어가 헷갈릴수는 있으나, VMM(virtual machine manager)를 보통 하이퍼바이저라고 부르고 vmm위에서 Dom0를 기동, 그리고 DomU를 그 위에 기동한다. 하테나에서 부르는 하이퍼바이저는 결국 Dom0이지만 관용적으로는 VMM을 가리킨다. 게스트 OS가 하이퍼바이저와 통신하기위해서는 원래 Dom0를 거쳐야 한다. 그런데 ParaVirtualization은 Hypercall이라는 특수 호출을 통해 하이퍼바이저와 바로 통신할 수 있고, resource를 더 빠르게 사용할 수 있다.
4. 리소스 소비 제어
   - 게스트 OS들이 사용하는 리소스를 소프트웨어 적으로 강력하게 제어할 수 있다. 



### 웹 서버와 DB 서버 간 가상화 구성의 차이

> 가상화 기술을 도입하는 가장 기본적인 목적은 하드웨어의 이용효율 향상이다. 하드웨어의 이용효율 향상을 위해 남아있는 리소스를 사용하는 게스트 OS를 투입한다.

- 웹 서버
  - 웹 서버의 경우 주로 CPU의 부하가 많이 발생하고 메모리의 부하는 다소 낮다. 따라서 메모리의 용량을 기존보다 늘리면 기존의 메모리에서 필요한 정도로 조금 늘리고, 나머지 남은 메모리를 캐시용 DB의 메모리로 가상화하여 사용할 수도 있다.
- DB 서버
  - DB 서버의 경우 메모리 부하가 잦다. 따라서 메모리의 용량을 기존보다 늘린다고 했을 때, 남는 I/O와 CPU의 리소스를 사용하기 위해 웹 서버를 추가하는 방안도 있다. 

*결국 상황에 따라 남는 리소스등을 파악하고 최대한 효율적으로 리소스를 사용할 수 있는 방안으로 관리하는 것이 핵심이다.*

#### 가상화로 얻은 장점

> - 물리적인 리소스 제약에서 해방
>
>   - 리소스를 동적으로 변경
>   - VM의 마이그레이션, 복제
>
>   -> 용이한 서버 증설 -> 더 나은 확장성 확보
>
> - 소프트웨어 레벨의 강력한 호스트 제어
>
>   - 비정상 동작 시 문제 국소화
>   - 호스트 제어가 용이해진다
>
>   -> 하드웨어 비용과 운용 비용 저하
>
>   -> 비용 대비 성능 향상, 고가용성으로 발전

#### 가상화와 운용

- 주로 서버 관리 툴은 각 서버가 탑재하고 있는 가상화 서버들의 Hierachy를 볼 수 있도록  되어있다. (아닐수도..)
- 또한 각 서버별로 부하를 확인할 때, 게스트 OS와 그 부모 OS의 부하를 함께 확인 할 수 있도록 명령어들을 커스터마이징 하는 경우도 있다.

#### 가상화의 단점

- 성능상의 오버헤드가 있다. (책이 쓰여진 2008년경의 경우)
  - CPU에서 2~3%의 오버헤드 발생
  - 메모리 성능에서 1할 정도의 오버헤드 발생
  - 네트워크 성능은 절반 정도
  - I/O 성능이 5% 정도 떨어진다.

*가상화는 정말 중요하다. 사실 처음에는 '가상화'라고 해도 와닿지 않았지만, 물리적인 공간을 논리적으로 변경하여 그 공간을 물리적인 것처럼 사용한다는 것은 정말 신기하다.*

### 하드웨어와 효율 향상

- 무어의 법칙(Moore's Law) 

  - 집적회로 상의 반도체의 성능은 24개월마다 2배로 증가한다. 

    -> 성능이 2배로 증가한 프로세서를 2년마다 같은 가격으로 구할 수 있다.

- *저가의 하드웨어들이라도 적절한 운용법만 따르면 효용성이 높다.*

  - RAID한다거나,
  - DISK가 없는 서버를 준비하거나
  - … etc






