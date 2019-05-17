## chapter 7 알고리즘 실용화 <sub>가까운 예로 보는 이론, 연구의 실전 투입</sub>



#### 알고리즘과 평가

#### 데이터 규모와 계산량 차이

- 데이터 내에서 원하는 결과를 찾는 선형 탐색의 경우 최악의 경우 데이터의 최대 수만큼 지속적으로 탐색해야 한다.
  - O(n)
- 하지만 이분탐색이라는 효율적인 알고리즘을 사용하면 반씩 쪼개어서 검색하기 때문에 절반, 그중의 절반, 그중의 절반, 그중의 절반 … 식으로 검색하기 때문에 최악의 경우 log n 만큼만 돌면 된다.
  - O(log n)



**이번 chapter에서는 두 가지 목적을 전제로 설명해 간다.**

1. 대규모 데이터를 앞둔 알고리즘 선택의 중요성을 느끼는 것
2. 좋은 알고리즘을 제품에 응용하고자 할 때, 실제로 알고리즘을 전개하기까지 어떠한 과정이 있는가?



#### 알고리즘이란?

- 좁은 의미 - 명확하게 정의된 계산문제를 정확하게 수행하는 것.
- 넓은 의미 - 도메인 로직 상의 흐름

- 알고리즘을 배우는 이점으로 가장 이해하기 쉬운 것은, 배워두면 새로운 문제에도 대처할 수 있다는 점이다.



#### 알고리즘의 평가 - Order 표기

- 입력크기가 n 일 때, 대략적으로 이정도의 계산량이 소요된다는 것을 표기하는 법. 
  - O(n), O(log n), O(n<sup>2</sup>) …. etc..
- 각종 알고리즘의 Order 표기 비교
  - O(1) < O(log n) < O(n log n) < O(n<sup>2</sup>) < O(n<sup>3</sup>) …. < O(2<sup>k</sup>)
- 일반적으로 O(log n)은 O(n)에 비해 상당히 빠르고, O(n)과 O(n log n) 사이에는 그다지 차이가 없으며, O(n)과 O(n<sup>2</sup>) 사이에는 계산이 끝나고 끝나지 않는 정도의 격차가 있다고 한다.



#### 알고리즘에 있어서 지수적, 대수적 감각

> 계산량이 지수적으로 증가하는 알고리즘은 이와 같이 데이터량이 적어도 계산량이 매우 커져버린다. 한편 '지수의 역인 대수적으로만 증가하는 O(log n)인 알고리즘은 데이터량이 꽤 커져도 적은 계산량으로 문제를 해결할 수 있다'는 것도 직감적으로 이해할 수 있을 것이다.



#### 알고리즘과 데이터 구조

- 알고리즘과 데이터 구조는 뗄래야 뗄 수 없는 관계이다. 데이터를 처리해야할 때, 적당한 데이터 구조를 선택하면, 처리를 단순화하고, 빠른 속도를 낼 수도 있다. 이는 데이터 구조가 가진 내부 알고리즘을 어느정도 이해하고 있어야 사용할 수 있다. 또한, 알고리즘에서 자주 사용하는 조작에 맞춘 데이터 구조를 선택할 필요성 또한 있다.
  - 데이터 구조를 잘 사용하기 위해서는 그 구조의 내부 알고리즘 정도는 이해해야 한다.
  - 데이터 처리를 위해 필요한 알고리즘에서 자주 사용하는 조작에 맞춰 데이터 구조를 선택할 필요가 있다.

#### 계산량과 상수항

- 일반적으로 오더 표기법은 상수항은 제외한다. 하지만 상수항이 많으면 당연히 속도는 떨어지게 된다. 
- 하지만 이공계적 관점에서 상수항은 항상 무시할만 한 양이라는 것을 직감적으로 느끼고 있다. 따라서, 상수항을 줄이기 위해 최적화에 신경을 쓰는 것보다는, 입력 데이터에 따른 계산량을 확 줄이는 방안으로 생각하는 것이 낫다.
  - O(n<sup>2</sup>) 의 상수항을 줄이기 보다 차라리 O(n log n)인 다른 알고리즘을 찾는다.



#### 알고리즘의 실제 활용

- 어떤 상황이든지 예측이나 측정이 중요하다. 때에 따라서는 명쾌하게 단순한 구현을 시도해보는 것도 좋다. 
- 좋은 알고리즘의 경우 많은 구현으로 오픈소스화 되어있다. 하지만 어느정도는 구현이 어떻게 되어있는지 알아두지 않으면 잘못된 선택을 할 수도 있다. 
  - 우리가 원하는 사양보다 오버스펙인 경우, 수정하기 위해서는 결국 알고리즘을 이해하고 있어야 한다.

#### 알고리즘을 대하는 자세

- Offensive한 자세 : 적극적으로 대규모 데이터를 응용하고, 그 결과에 따라 애플리케이션에 부가가치를 추가하는 것
- Defensive한 자세 : 대규모 데이터를 빠르게 정렬하거나 검색, 압축하는 일은 발생하는 문제를 얼마나 잘 맞아들이는가

>대규모 데이터에 대해 알고리즘 측면에서의 접근법을 배우려고 할 때에는 기존 방법은 어느 정도 자신의 지식으로서 익혀두는 것이 중요하다. 키워드 링크에 Trie를 응용할 수 있다는 발상은 Trie가 어떤 데이터 구조인지 그 특성을 모른다면 생각해낼 수 없을 것이다. 또한 이런 알고리즘을 구현한 후에도 실용화까지는 어느 정도의 추가작업이 필요하다는 것은 앞에서도 얘기한 대로다.





_ _ _

8장, 9장, 10장 실습으로 정리 제외\

_ _ _






