## chapter6. [과제] 압축 프로그래밍 <sub>데이터 크기, I/O 고속화와의 관계 인식하기</sub>

- 대규모 데이터를 다루는 서비스에서 데이터의 압축으로 얼만큼의 효과를 얻을 수 있는지 과제를 통해 깨달아 본다. 
- 압축은 I/O의 고속화와는 끊을래야 끊을 수 없는 관계이다.
- 속도에 주의를 기울인다고는 해도 실제로 프로그램이나 알고리즘에서 얼마나 빠른지에 대한 감각도 중요하다.
- 데이터형의 크기, 바이너리의 크기 혹은 스크립트 언어가 갖는 데이터 크기의 오버헤드 등이 데이터 압축에서 갖는 중요한 의미를 깨달아야 한다.



### 정수형 데이터를 컴팩트하게 가져가기

- 보통 정수형 데이터는 4Byte(32bit) 이다. 그럼 만약 5라는 숫자를 컴퓨터에서 표현하면 아래와 같이 32bit로 표현할 수 있다. 우리가 정작 필요로 하는 것은 맨 앞의 101<sub>(2)</sub> 뿐이다. 

  

![스크린샷 2019-05-03 오후 3.54.04](/Users/noah/Desktop/스크린샷 2019-05-03 오후 3.55.37.png)

- 결국 엄밀하게 말하자면 5라는 숫자를 표현하기 위해서는 아래의 그림과 같이 딱 1byte의 데이터 길이만 필요하다는 것이다.





![캡처](/Users/noah/Desktop/스크린샷 2019-05-03 오후 3.56.09.png)

**책에서 다루고 있는 VBCode의 정수형 데이터 압축 개념은 기본적으로 위의 개념을 큰 틀로 한다. 결국 필요없는 데이터인 오버헤드를 줄임으로써 데이터를 압축할 수 있는 것이다. (데이터를 Compression 했으면, 컴퓨터가 이해할 수 있는 원본 데이터로 Depression 하는 기능도 구현해야만 한다.)**



#### Gap

- 정수형 데이터의 압축(혹은 모든 데이터의 압축)의 주요 개념은 정수의 Gap 값을 가져간다는 것인데, 

  - [3, 5, 20, 21, 23, 76, 77, 78]

    —> [3 ,2, 15, 1, 2, 53, 1, 1]

  이 된다는 말이다. 결국 원래 데이터보다 적은 수의 데이터이기 때문에 그만큼 생기는 오버헤드를 VBCode 알고리즘을 적용하여 줄일 수 있다. 



#### 압축의 기초

> 기호(데이터)의 출현빈도를 보고 빈번하게 출현하는 기호에 짧은 부호를 부여하고 그렇지 않은 기호에는 긴 부호를 부여한다. 즉 기호의 출현확률분포를 생각해서 최적인 부호를 생성한다. 이것이 압축의 가장 근저에 있는 이론이다. VBCode에서는 Gap을 취함으로써 작은 숫자가 많이 나타나기 쉽도록 확률분포를 만들고, 이에 대해 작은 정수에 짧은 부호를 부여함으로써 압축의 기본이론에 충실하고 있다. -> 모스 신호와 같은 원리



#### 다양한 압축 알고리즘

- 정수형 데이터의 압축
  - Unary code
  - Gamma code
  - Delta code
  - VB code (Variable Byte code)
  - etc...
- 문자열 데이터의 압축
  - Huffman code
  - etc...

#### 마지막으로 주의할 점

- 압축은 어디까지나 데이터를 특정 알고리즘을 활용한다. 이 말은 알고리즘이 충분한 속도를 보장할 때에, 그리고 운용중인 서비스에 적용하여 속도의 개선이 이루어질 때 사용해야 한다는 말이다. 
  - 만약, (기존 I/O 시간)  <  (압축에 드는 CPU 계산 시간 + 이를 통해 압축된 데이터의 I/O 시간) 이라면, 굳이 압축을 사용하지 않아도 괜찮다는 말이다.(물론 추후에 더욱 서비스가 커진다면 고려해야겠지만..)
  - 결국 "정답은 없다" 가 아닐까? 애플리케이션이 서비스되기 전에 애초에 개발단계에서 너무 많은 사항에 대해 고려한다는 것 또한 리소스의 낭비일 수 있다. 문제에 직면한다면 적절히 해결할 수 있는 능력이 중요하지 않을까?









## VBCode 실습 (java)

### 목표

- **VBCode를 활용하여 정수형 데이터를 압축하는 것이 얼마나 데이터의 양을 줄일 수 있는지 직접 확인한다.**

- **depression이 제대로 동작하는지 확인한다.**

  

### 과정

1. 단순히 1부터 순차적으로 1씩 증가하는 정수열을 out.txt 파일에 써둔다. 데이터는 매 13번째 정수마다 \n을 입력하여 br.readLine()이 메모리 부족을 일으키지 않도록 한다.
2. 데이터를 얼추 넣어 총 204MB의 크기를 갖는 out.txt파일을 생성하였다. 이를 아래의 encode 로 압축하여 encoded.txt라는 파일에 byte 로 쓴다.
3. 다시 encoded.txt를 읽어들여, depression을 진행하였을 때, 데이터가 정확히 복구되는지 확인한다.

아래는 VBCode.java이다[^1] 

~~~java
import java.nio.ByteBuffer;
import java.util.ArrayList;
import java.util.List;

public class VBCode {
	 private static byte[] encodeNumber(int n) {
	        if (n == 0) {
	            return new byte[]{0};
	        }
	        int i = (int) (Math.log(n) / Math.log(128)) + 1;
	        byte[] rv = new byte[i];
	        int j = i - 1;
	        do {
	            rv[j--] = (byte) (n % 128);
	            n /= 128;
	        } while (j >= 0);
	        rv[i - 1] += 128;
	        return rv;
	    }

	    public static byte[] encode(List<Integer> numbers) {
	        ByteBuffer buf = ByteBuffer.allocate(numbers.size() * (Integer.SIZE / Byte.SIZE));
	        int last = -1;
	        for (int i = 0; i < numbers.size(); i++) {
	            Integer num = numbers.get(i);
	            if (i == 0) {
	                buf.put(encodeNumber(num));
	            } else {
	                buf.put(encodeNumber(num - last));
	            }
	            last = num;
	        }

	        buf.flip();
	        byte[] rv = new byte[buf.limit()];
	        
	        
	        buf.get(rv);
	        return rv;
	    }

	    public static List<Integer> decode(byte[] byteStream) {
	        List<Integer> numbers = new ArrayList<Integer>();
	        int n = 0;
	        int last = -1;
	        boolean notFirst = false;
	        for (byte b : byteStream) {
	            if ((b & 0xff) < 128) {
	                n = 128 * n + b;
	            } else {
	                int num;
	                if (notFirst) {
	                    num = last + (128 * n + ((b - 128) & 0xff));

	                } else {
	                    num = 128 * n + ((b - 128) & 0xff);
	                    notFirst = true;
	                }
	                last = num;
	                numbers.add(num);
	                n = 0;
	            }
	        }
	        return numbers;
	    }
}

~~~



## 결과

- 사실 이 실습 전에 예행으로 진행한 적이 있는데, byte로 저장하지 않고 나온 GAP의 byte값을 그대로 String으로 저장하는 바보같은 짓을 하였다. 결국 모든 숫자는 인접한 수끼리 1씩 차이나므로 -127이 저장되었고, 이를 String으로 출력하였으므로, 단순한 String의 길이 변화에 따라 Compression 된 것 뿐이었다. 그 때는 204MB의 파일이 대략 134MB의 파일로 Compression되었다. 하지만 InputStream과 OutputStream을 활용하여 byte단위로 압축하였을 때는, 놀라운 결과가 나왔다.
총 50MB의 크기를 가진 파일을 압축 시도했고, <br>
![스크린샷_2019-05-06_오후_9.10.05](uploads/fbb61d0dfc73e75497faa1ef66800075/스크린샷_2019-05-06_오후_9.10.05.png)


결국 7.7MB의 파일로 압축할 수 있었다. 비율로 따지면 놀라운 결과이다. 한번의 실습을 통해, 압축의 효율을 몸소 느낄 수 있었다.<br>

![스크린샷_2019-05-06_오후_9.08.03](uploads/ce5adaf0aa25e4c68ddf7c2d328aef97/스크린샷_2019-05-06_오후_9.08.03.png)




## 의문점
- 책의 설명에 따르면 4byte의 데이터를 필요없는 오버헤드를 삭제한 byte로 바꾸어서 저장하는 것이다. ~~그런데 이 코드의 경우 java가 자동으로 byte를 오버헤드를 자른 체로 encoded.txt에 저장한 것인지 정확히 이해할 수 없었다.~~[^2]








_ _ _

[^1]: [https://gist.github.com/zhaoyao/1239611](https://gist.github.com/zhaoyao/1239611) 의 코드를 활용하였으며, 필요한 부분은 수정하여 사용하였다.
[^2]: 1바이트 단위씩 파일을 읽고 쓰는 FileInputStream과 FileOutputStream을 활용하는 것으로 해결할 수 있었다.
_ _ _

[^1]: [https://gist.github.com/zhaoyao/1239611](https://gist.github.com/zhaoyao/1239611) 의 코드를 활용하였으며, 필요한 부분은 수정하여 사용하였다.