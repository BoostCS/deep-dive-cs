## Garbage Collector
> 동적으로 할당된 메모리 영역 중 사용하지 않는 영역을 탐지하여 해제하는 기능
- 동적으로 할당했던 메모리 -> 힙 메모리
- 사용하지 않는 영역           -> 어떤 변수도 가리키지 않는 영역

### ❓GC가 왜 필요할까?
- 메모리 누수 방지
- 해제된 메모리 접근 방지
- 해제한 메모리 또 해제 방지 -> 이중 해제 방지

==그럼 GC에는 단점이 없는걸까?==
> 어느 프로그램이든 단점은 존재할 수 밖에 없다.
- GC 작업은 순수 오버헤드
	- 메모리 영역이 해제의 대상이 될지 검사하고 해제하는 일은 프로그램이 해야 할 일을 못하도록 방해하기 때문이다.
- 개발자는 GC가 메모리를 해제하는지 모른다.
**JVM에서 GC가 알아서 메모리 영역을 해제 해준다고 해서 우리가 메모리를 신경쓰지 않아도 되는 것은 아니다!**

## 🧐 GC 알고리즘
> Reference Counting과 Mark And Sweep에 대해서만 정리

### Reference Counting
- 참조횟수 == 0 은 GC의 대상이 된다.
- 참조횟수 >= 1 은 GC의 대상이 되지 않는다.

![ReferenceCounting1](https://user-images.githubusercontent.com/53300830/208671478-eb8a8f67-b96e-4687-b32e-15db498582e9.png)

- 스택변수 전역변수 등 heap 영역 참조를 담은 변수라고 생각하면 편하다

![ReferenceCounting2](https://user-images.githubusercontent.com/53300830/208671528-85f6dc85-f215-40b7-9e5d-67ed18fa78c9.png)

- Heap 영역에 선언된 객체들이 각각 reference count라는 별도 숫자를 가지고 있다고 생각하면 이후 GC에 대해서 더 편하게 이해가 가능하다!
- 여기서 reference count는 몇 가지 방법으로 해당 객체에 접근할 수 있는지를 뜻한다.
- 하나도 없다면 0이 되고, GC에 대상이 되는 것이다.

![ReferenceCounting3](https://user-images.githubusercontent.com/53300830/208671573-e50a12b0-efbc-4172-bb76-a755ab08cf7f.png)


- 하지만 이런 상황이면 어떻게 될까? 서로 참조를 하고 있는 순환구조인 상태이다.
- Root Space 에서의 접근을 모두 끊었지만 네모 안에는 서로가 참조를 하고 있기 때문에 Reference Count는 1이 된다.
**사용하지는 않지만 GC에서는 얘 사용 중이야! 하는 것으로 오해를 하고 있기 때문에 Memory Leak 가 생기게 됩니다.**

### Mark-Sweep
> Reference Counting의 순환 참조 문제를 해결해준다.
> 루트에서부터 해당 객체에 접근이 가능한지 불가능한지가 기준이 된다.

![MarkAndSweep1](https://user-images.githubusercontent.com/53300830/208671673-8ca96286-46d8-4c62-86ea-2e242acde8fc.png)
- 루트부터 그래프를 순회해서 연결된 객체들을 찾아내고, 연결되지 않은 객체들은 지우게 된다.
- 루트로부터 연결된 객체는 : Reachable
- 루트로부터 연결이 끊긴 객체는 : UnReachable

![MarkAndSweep2](https://user-images.githubusercontent.com/53300830/208674028-b82ea6fd-0d2c-464d-b417-caeefeb11f64.png)
- 분산돼 있던 메모리가 정리된 것을 볼 수 있다.
- 이런 메모리 파편화를 막는 것을 Compaction이라고 한다.
- Java와 JS 둘 다 이 방식을 사용한다.

==Mark-Sweep 에는 단점이 없는걸까?==
> 언제나 그렇듯 장점이 있으면 단점이 있다.
- 의도적으로 GC를 실행시켜야 한다. (Reference Counting 과 반대이다.)
- 어플리케이션 실행과 GC 실행이 병행된다
**이처럼 어플리케이션의 사용성을 유지하면서 효율적이게 GC를 실행시키는 것은 엄청나게 어려운 최적화 작업이다.**

### Mark Sweep 예시

**크게 Young Generation과 Old Generation으로 나뉘어진다. 편의상 Y G로 설명하겠습니다.**

-   Y에서 발생하는 GC는 `minor gc`
-   O에서 발생하는 GC는 `major gc`
    -   세 영역으로 나눠어진다.
    -   Eden은 새롭게 생성된 객체
    -   `Survival` 은 `minor gc` 로 부터 살아 남은 객체들이 존재한다.
        -   이때 신기한 규칙이 있다. `survival0` 혹은 `survival1` 은 비어있어야만한다.

![Mark_Sweep_1](https://user-images.githubusercontent.com/53300830/208671613-952fca61-066e-4d93-8f18-f5f8c177d18e.png)
> 위 초록색은 루트로부터 Reachable(살아남을 객체들) 노란색은 UnReachable(해제될 객체들)이라 판단된 객체입니다.

-   이렇게 Reachable이라 판단된 객체는 Survival로 옮겨지게 됩니다.


![Mark_Sweep_2](https://user-images.githubusercontent.com/53300830/208671620-ed4b48a7-7bd9-46d5-b28f-9e237ab6a6de.png)
-   Survival 0으로 옮겨진 객체들의 숫자가 1로 바뀌게 된다. (그리고 노란색 객체들을 참조가 해제됩니다.)
-   해당 숫자는 `age-bit` 라고 한다. `major gc` 에서 살아남을때마다 `+1` 이 된다.


![Mark_Sweep_3](https://user-images.githubusercontent.com/53300830/208671622-5a25b1f6-e50a-4e68-9f96-15a8a6a6f57a.png)
- 시간이 흘러 Eden에 또 꽉차면 Reachable이라 판단된 객체들이 Survival1로 옮겨지게 된다.


![Mark_Sweep_5](https://user-images.githubusercontent.com/53300830/208671624-ffd03486-c908-4e57-b9a3-4f646e0d7ddd.png)
-   이러한 과정 속에서 `age-bit` 가 3이 된 객체가 있다고 가정하겠습니다.
-   일정 수준의 `age-bit` 를 넘어가면 오래도록 참조될 객체라는 것으로 판단하여 `Old Generation`으로 옮겨가게 됩니다. 이것을 `promotion` 이라고 부른다.
    -   Java 8기준으로는 `age-bit`가 15가 되면 `promotion`이 진행이 된다.

==이러한 과정속에서 Old Generation도 언젠가는 꽉차게 된다.==
-   이때 `major GC`가 발생한다!
-   똑같이 Mark And Sweep 방식을 통해 필요 없는 메모리를 비우게 된다.
    -   당연하게도 크기가 크기 때문에 `minor gc` 보다 오래 걸린다!

## 😀 Old 영역에 대한 GC
> Old 영역은 기본적으로 데이터가 가득 차면 GC를 실행한다.
> 이때 방식에 따라서 절차가 달라지므로, 어떤 GC가 있는지 정리를 해보자!
- Serial GC
- Parallel GC
- Parallel Old GC(Parallel Compacting GC)
- Concurrent Mark & Sweep GC (CMS)
- G1 (Garbage First) GC

==운영 서버에서 절대 사용하면 안되는 방식은 Serial GC이다.==
- Serial GC는 데스크톱 CPU 코어가 하나만 있을 때 사용하기 위해서만든 방식이다.
- Serial GC를 사용하면 애플리케이션의 성능이 많이 떨어진다.

### Serial GC
>Young 영역에서의 GC는 앞 절에서 설명한 방식을 사용한다. 
- Old 영역의 GC는 mark-sweep-compact 알고리즘을 사용한다.
- 첫 단계는 Old 영역에 살아 있는 객체를 식별하는 것이다.
- 그 다음에 힙의 앞 부분부터 확인하여 살아있는 것만 남긴다.
- 마지막 단계에서는 각 객체들이 연속되게 쌓이도록 힙의 가장 앞 부분부터 채워서 객체가 존재하는 부분과 객체가 없는 부분으로 나눈다. (Compaction)

### Parallel GC
>Parallel GC는 Serial GC와 기본적인 알고리즘은 같다. 
- 그러나 Serail GC는 GC를 처리하는 스레드가 하나인 것에 비해, Parallel GC는 GC를 처리하는 스레드가 여러 개이다.
- 그렇기 때문에 Serail GC 보다 빠르게 객체를 처리할 수 있다.
- Parallel GC는 메모리가 충분하고 코어의 개수가 많을 때 유리하다.

![SerialVsParallel](https://user-images.githubusercontent.com/53300830/208671860-28f23b6d-8d31-46fc-b009-01795c7a17fe.png)

### Parallel Old GC
> 앞서 설명한 Parallel GC와 비교하여 Old 영역의 GC 알고리즘만 다르다.
- Mark-Summary-Compaction 단계를 거친다.
- Summary 단계는 GC를 수행한 영역에 대해서 별도로 살아 있는 객체를 식별한다는점에서 Mark-Sweep-Compaction 알고리즘의 Sweep 단계와 다르며, 아주 약간 더 복잡한 단계를 거친다.

### CMS GC
>초기 Mark 단계에서는 클래스 로더에서 가장 가까운 객체 중 살아 있는 객체만 찾는 것으로 끝낸다. 
>따라서 멈추는 시간이 매우 짧다.
- Concurrent Mark 단계에서는 방금 살아있다고 확인한 객체에서 참조하고 있는 객체들을 따라가면서 확인한다.
	- 다른 스레드가 실행중인 상태에서 동시에 진행된다는 것이다.
- Remark 단계에서는 Concurrent Mark 단계에서 새로 추가되거나 참조가 끊긴 객체를 확인한다.
- Concurrent Sweep 단계에서는 쓰레기를 정리하는 작업을 실행한다.
	- 이 작업도 다른 스레드가 실행되고 있는 상황에서 진행한다.

![CMS](https://user-images.githubusercontent.com/53300830/208671907-bf7a495a-b7ee-4a58-bf36-9f0603380b16.png)

==하지만 확실한 단점이 존재한다.==
- 다른 GC 방식보다 메모리와 CPU를 더 많이 사용한다.
- Compcation 단계가 기본적으로 제공되지 않는다.

> 문제가 많아서 그런지 Java9부터는 Deprecated 됐고, Java 14버전부터는 사용이 중단됐다.

### G1 GC
> G1 GC를 이해하려면 지금까지 이해하고 있던 Young 영역과 Old 영역을 잊어버려야 한다.
- G1 GC는 개념적으로 그들이 존재하나 일정 크기의 논리적 단위인 region으로 구분하고 있다.
- Humonogous: Region 크기의 50%를 초과하는 큰 객체를 저장하기 위한 공간
- Available/Unused: 아직 사용되지 않은 Region

![G1GC](https://user-images.githubusercontent.com/53300830/208671981-185c3b53-9580-4f2c-ad62-9be304c30933.png)
- 해당 영역이 꽉 차면 다른 영역에서 객체를 할당하고 GC를 실행한다.
- 지금까지 설명한 Young의 세가지 영역에서 데이터가 Old 영역으로 이동하는 단계가 사라진 GC 방식으로 이해하면 된다.

## 🧐 G1 GC에 대해서 조금만 더 깊게 들어가보자!

### G1 GC의 Cycle
![G1GC의Cycle](https://user-images.githubusercontent.com/53300830/208672033-73fc111c-6715-4ff8-911e-5e50624b7b00.png)

- Young-Only와 Space Reclamation를 반복한다.
- 사이클 중 모든 원은 STW가 발생한 것을 나타낸 것이고, 원의 크기에 따라 STW 소요 시간이 달라진다고 생각하면 된다!
### Young Only
> Young Only는 Minor GC만 수행하다가 Old Generation 비율에 지정된 값이 초과하는 순간 Major GC가 수행된다.

- Major GC의 첫 단계는 Initial Mark이며, Minor GC와 동시에 수행되며 둘 다 STW를 수반한다.
- 애플리케이션 스레드, Minor GC, Concurrent Mark가 동시에 수행되는데 Remark가 수행되는 순간 다른 작업은 멈추게 된다.
	- 그래서 Remark에 해당하는 주황색 원이 큰 것이다.
- 그 이후 Minor GC가 수행되다가 Major GC의 Cleanup이 발생한다.

### Space Reclamation
> Young Only가 끝나고 Space Reclamation이 시작된다.

- 해당 Phase에서는 Mixed GC가 수행된다.
	- Mark 단계가 없어서 STW(Stop The World) 빈도가 Young Only Phase에 비해 줄어든다.
	- Space Reclamation이 끝나면 다시 Young Only로 돌아가서 Minor GC를 실행한다.

### G1 GC Minor GC(Young Generation) 동작 과정
![G1_MInor_1](https://user-images.githubusercontent.com/53300830/208672082-ce974ce0-13a8-42eb-8139-237e4aac343c.png)
- 연속되지 않은 메모리 공간에 Young Generation이 Region 단위로 메모리에 할당된다.

![G1_MInor_2](https://user-images.githubusercontent.com/53300830/208672091-e73e2bad-a5d9-4129-90b6-494258feef9f.png)
- Young Generation에 있는 유효 객체를 Survivor Region이나 Old Generation으로 copy or move한다.
- 이 단계에서 STW가 발생하고, Eden Region과 Survivor Region의 크기는 Minor GC를 위해 다시 계산된다.

![G1_MInor_3](https://user-images.githubusercontent.com/53300830/208672096-ef5fd112-182c-4bc7-9783-52ac16286572.png)

- Minor GC를 모두 마친 후의 모습이다.
- 청록색 영역은 Eden 영역에서 Survivor 영역으로 이동하거나 Survivor에서 Survivor로 이동한 것을 뜻한다.

### G1 GC Major GC(Old GC) 동작 과정
> Minor GC와 원리가 비슷하나 멀티 스레드에서 병렬로 동작한다.

![G1_Major_1 1](https://user-images.githubusercontent.com/53300830/208673430-7b9c5bf2-939f-4746-ab2b-6268b4338256.png)
- Initial Mark 단계는 Old Region에 존재하는 객체들이 참조하는 Survivor Region이 있는지 파악 후 Survivor Region에 마킹하는 단계이다.
- Survivor Region에 의존하기 때문에 Survivor Region은 깔끔한 상태여야 하고, Survivor Region이 깔끔하려면 Minor GC가 전부 끝난 상태여야 한다.
- 따라서 Inital Mark는 Minor GC에 의존적이며, STW를 발생한다.
- Inital Mark 이후 Root Region Scan 단계가 수행된다.
- 마킹된 Survivor Region에서 Old Region에 대해 참조하고 있는 객체를 마킹한다.
- 멀티 스레드로 동작하며 다음 Minor GC가 발생하기 전에 동작을 완료한다.


![G1_Major_2 1](https://user-images.githubusercontent.com/53300830/208673547-9cdbb1a9-684c-41b2-bf2a-261e904ba983.png)
- Concurrent Marking 단계에서는 Old Generation 내에 생존해 있는 모든 객체를 마킹한다.
- STW가 발생하지 않으므로 애플리케이션 스레드와 동시에 동작하고, Minor GC와 같이 진행하므로 Minor GC에 의해 중단될 수 있다.
- 사진에서 X 표시된 Region은 모든 객체가 Garbage 상태인 영역이다.


![G1_Major_3 1](https://user-images.githubusercontent.com/53300830/208673666-5ff68c55-0dfa-46f6-b9c5-de6f074de1e8.png)
- Remark 단계는 Concurrent Mark 단계에서 X표시된 영역을 회수한다. -> STW 발생
	- Mark 단계를 마저 끝낸다.
	- 이때 SATB(Snapshot-At-The-Beginning) 기법을 사용하기 때문에 CMS GC보다 더 빠르다.
	- SATB는 STW 이후 살아있는 객체에만 마킹하는 알고리즘이다.


![G1_Major_4](https://user-images.githubusercontent.com/53300830/208673707-2142f5d1-e8f7-4d79-8ff9-b3b5c00df009.png)
- Copying/Cleanup 단계에서는 STW가 발생하며 Live Ojbect의 비율이 낮은 영역 순으로 순차적으로 수거해간다.
- 먼저 해당 영여게서 Live Object를 다른 영역으로 move or copy한 후 Grabage를 수집한다.
- G1 GC는 이렇게 Garbage의 수집을 우선해서 계속하며 여유 공간을 신속하게 확보해 둔다.


![G1_Major_5](https://user-images.githubusercontent.com/53300830/208673743-4056040d-8ef9-4beb-9b94-1fd02beeaf5f.png)
- Major GC가 끝난 이후 Live Object가 새로운 Region으로 이동하고 메모리 Compaction이 일어나서 깔끔해진다.

### Mixed GC
- Young 영역과 Old 영역의 Garbage를 수집한다. 한 번에 Old 영역의 Garbage를 수집하는 것은 비용이 크므로 Mixed GC는 기본적으로 8회 수행된다.
- Minor GC에서 수행하는 단계와 동일하지만, 추가로 Old 영역의 Garbage를 수집한다. 즉, Mixed GC는 Minor GC와 Old 영역의 GC를 혼합한 과정이라고 할 수 있다.

## 📋 GC가 면접 질문으로 들어온다면
- GC가 무엇인지, 어떤 알고리즘을 쓰는지는 기본적으로 대답을 해야 한다.
- 가능하다면 직접 GC 모니터링을 해본 경험 까지 말하는 것을 추천하다.
	- SCOUTER, Jenifer -> 성능 모니터링 오픈소스 
	- 안드로이드는 Perfetto를 사용한다.
	- dump 분석해본 후 -> GC 분석도구 한 번쯤은 해보는게 좋다.

> 개발자는 운영능력도 중요하다. GC는 초반 질문으로도 충분히 가능하다.
> 안드로이드 개발자가 백엔드는 아니기에 엄청 많은 지식을 원하지는 않겠지만 이정도 언급을 한다면 초반에 좋은 인상을 줄 수 있을 것이다.

### 메모리 사용량 테스트

```kotlin
object Util {  
    const val max = 5_000_000  
    private val rt: Runtime = Runtime.getRuntime()  
  
    fun printHeap(idx: Int) {  
        rt.gc()  
        val total = rt.totalMemory() // 메모리 총 합  
        val free = rt.freeMemory() // 사용 가능한 메모리  
        val used = total - free // 사용 중인 메모리는 전체에서 사용가능한 메모리를 뺀 값  
        println(String.format("%d Heap:%, 8d bytes", idx, used))  
    }  
}

fun main() {  
    Util.printHeap(0)  
  
    val sb = StringBuffer()  
    for (i in 0..Util.max) {  
        sb.append("a")  
    }    
    Util.printHeap(1)  
    Util.printHeap(2)  
  
    /**  
     0 Heap: 457,536 bytes     
     1 Heap: 19,476,416 bytes     
     2 Heap: 19,474,376 bytes     
     */
}
```

```kotlin
/**  
 * 메모리단에서 String으로 갖고 있느냐 byte 상태로 갖고 있느냐에 따라서 두 배 정도 차이가 있다. 
 * 파일에 데이터를 엑세스하거나 할 때 참고할만한 자료이다! 
 * */
fun main() {  
    Util.printHeap(0)  
  
    var sb: StringBuffer? = StringBuffer(Util.max)  
    for (i in 0 until Util.max) {  
        sb?.append("a")  
    }    
    val b = sb.toString().toByteArray()  
    sb = null // StringBuffer null 처리해서 gc에서 날라가게 설정  
    Util.printHeap(1)  
    Util.printHeap(2)  
  
    /**  
     0 Heap: 432,120 bytes     
     1 Heap: 5,601,936 bytes     
     2 Heap: 5,599,440 bytes     
     */
}
```



### 질문 예시
- GC가 무엇이고 장단점은 어떻게 되는지?
- GC 알고리즘은 어떤게 있는지
- GC의 Survivor이 왜 두 개인지
- Mark-Sweep 방식에서 GC가 실행되는 타이밍은 언제인지
- GC의 실행 방식을 설명해주세요
- G1 GC가 무엇이고 장단점이 어떤게 있나?
- G1 GC의 Heap 구조를 설명해주세요.
- G1 GC의 동작 과정을 설명하라.

### 참고 자료 
https://rebelsky.cs.grinnell.edu/Courses/CS302/99S/Presentations/GC/
https://d2.naver.com/helloworld/1329
https://d2.naver.com/helloworld/37111
https://perfectacle.github.io/2019/05/11/jvm-gc-advanced/
https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html
https://steady-coding.tistory.com/591
