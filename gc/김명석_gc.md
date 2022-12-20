## JVM 
---
- 운영체제 메모리 영역에 접근하여 메모리를 관리하는 프로그램  
- 메모리 관리, Garbage Collector 역활 수행

### JVM 구성 요소
![JVM 구성요소](https://user-images.githubusercontent.com/9216335/208658204-7f72ed4c-4f9c-4070-895c-bb2a2b9876b3.png)

**Class Loader**  
바이트코드를 읽고, 클래스 정보를 메모리의 Heap/Method Area에 저장  
**Memory**  
실행 중인 프로그램의 정보가 올라가 있는 메모리
- Heap/Method Area : 모든 쓰레드가 공유하는 영역
  - Method Area : 클래스 구조를 메타데이터 처럼 가지고 있으며 메서드의 코드를 저장
  - Heap : 어플리케이션 실행 중에 생성되는 객체 인스턴스를 저장
- Stack/PC Register/Native Method Stack : 각 쓰레드마다 고유하게 생성 쓰레드 종료 시 소멸
  - Stack : 메서드 호출을 스택 프레임이라는 블록으로 쌓으며 로컬 변수, 중간 연산 결과들이 저장
  - PC Register : 쓰레드가 현재 실행할 Stack Frame의 주소를 저장
  - Native Method Stack : C/C++ 등의 Low level 코드를 실행하는 스택

**Execution Engine**  
바이트코드를 네이티브 코드로 변환, GC 를 실행

## GC 
---
- 동적으로 할당한 메모리 영역 중 사용하지 않는 영역을 탐지하여 해제하는 기능  
- JVM의 Heap 영역에서 사용하지 않는 객체를 삭제하는 프로세스

### GC가 왜 필요한지
> 동적으로 할당 했던 메모리중 필요없게 된 영역을 알아서 해제
- 장점
  - 메모리 누수를 막을 수 있다
  - 해제된 메모리에 접근을 막을 수 있다
  - 해제한 메모리를 다시 해제하는 상황을 막을 수 있다
- 단점
  - GC 작업은 순수 오버헤드
  - 개발자는 언제 GC가 메모리를 해제하는지 모른다

### GC 설계자들의 가설
- 대부분의 할당된 객체는 오랫동안 참조되지 않으며 금방 Garbage 대상이 된다
- 오래된 객체에서 젊은 객체로의 참조는 거의 없다

- 스캔비용을 줄이기위해 영역을 나눈다 

## GC 알고리즘
---
### Reference Counting
- 몇 가지 방법으로 해당 객체에 접근 할 수 있는지
- 0 이 되면 해제
  - 순환 참조의 경우 메모리 릭이 발생
### Mark And Sweep
![MarkAndSweep Root Set](https://user-images.githubusercontent.com/9216335/208663141-315a1646-dbcf-4c82-b688-d8b4d950778a.png)
- Reference Counting 의 순환 참조 문제를 해결
- Root Space 에서 해당 객체에 접근이 가능한지
- Mark
  - 그래프 순회를 통해 연결된 객체(Reachable)를 찾아내 마킹한다
- Sweep
  - 연결되지 못한 객체(Unreachable)를 제거
- Compaction
  - 메모리 파편화를 막기위해 진행, 안하는 경우도 있음  
- Young generation (minor gc)
  - eden / survival0 / survival1
  - eden : 새롭게 생성되는 객체들이 할당되는 영역
  - survival 영역은 minor gc로 부터 살아남은 객체들이 존재하는 영역
  - survival 0/1은 둘 중 하나는 비어 있어야 한다
  - 살아 남을때 마다 age-bit 가 1씩 증가
- Old generation (major gc)
  - Old Generation 이 가득차면 발생 
  
**단점**
- 의도적으로 GC를 실행시켜야한다
- 어플리케이션 실행과 GC의 실행이 병행된다

### Root Space
- Stack의 로컬 변수
- Method Area 에 저장된 static 변수
- Native Method Stack 의 C/C++로 작성된 JNI 참조

### Stop-The-World
- GC를 실행하기위해 JVM이 애플리케이션 실행을 멈추는 것
- stop-the world가 발생하면 GC를 실행하는 스레드를 제외한 나머지 스레드는 모두 작업을 멈춘다
- GC 작업을 완료한 이후에 중단한 작업을 다시 시작한다.

## Garbage Collector 종류
---
### Serial GC
![image](https://user-images.githubusercontent.com/9216335/208672185-d8a8475b-1f5a-430d-89ee-b65cc835ba2d.png)
- GC를 처리하는 스레드가 1개이다
- CPU 코어가 1개만 있을 때 사용하는 방식
- Mark-Compact collection 알고리즘 사용
  - Mark And Sweep 과정 진행 후 메모리를 한 곳으로 모아서 파편화를 방지한다
- Stop The World 시간이 긺
- 싱글 쓰레드 환경 및 Heap이 매우 작을 때 사용

### Parallel GC (Java 8의 Default)
![image](https://user-images.githubusercontent.com/9216335/208672149-4cb96974-4039-4306-9322-2298394ba837.png)
- GC를 처리한는 스레드가 여러 개 이다
- Serial GC보다 빠르게 객체를 처리할 수 있다
- Parallel GC는 메모리가 충분하고 코어의 개수가 많을 때 사용하면 좋다
- Java 8 의 Parallel GC 기준 age-bit 15가 되면 Promotion 진행

### Concurrent Mark Sweep GC
![image](https://user-images.githubusercontent.com/9216335/208671721-c83ee69e-ba6d-476c-ba27-a164391c8b22.png)
- Stop the world 시간을 줄인 GC 
- 응답시간이 빨라야할 때 CMS GC를 사용한다
- 다른 GC 방식보다 메모리와 CPU를 더 많이 사용한다
- GC 작업과 어플리케이션을 동시에 실행
- Compaction(압축) 단계가 제공되지 않는다  
- G1 GC 등장에 따라 Deprecated

**과정**
- Initial Mark : GC Root가 참조하는 객체를 찾아 마킹한다
- Concurrent Mark : 애플리케이션 동작과 동시에 Reachable Object가 참조하고 있는 객체를 찾아 마킹한다
- Remark : 애플리케이션 실행도중 생긴 객체를 다시 Mark 과정을 거친다
- Concurrent Sweep : Unrechable Object 를 Sweep 한다 (Stop the world 가 발생)
  
### G1 GC
![image](https://user-images.githubusercontent.com/9216335/208672346-fb0cd379-f6ee-41a7-aabb-f2849082f5dd.png)
- 각 영역을 Region으로 나눈다
- GC가 일어날 때, 전체 영역을 탐색하지 않는다.
- 바둑판의 각 영역에 객체를 할당하고 GC를 실행한다
- 해당 영역이 꽉차면 다른 빈 영역에 객체를 할당하고 GC를 실행한다
- G1 GC는 Stop The World 시간이 짧다
- Compaction을 사용한다
- Java9 부터 default GC 방식

## JVM GC 튜닝 맛보기
---
> 코드레벨에서 튜닝이 이루어지고 하는 최종 단계의 튜닝
- 목표
  - Old Generation 으로 넘어가는 객체 최소화하기
  - Major GC 시간을 짧게 유지하기
  - 메모리가 크다면 가끔 일어나지만 오래걸림
  - 메모리가 작다면 자주 일어나지만 금방 끝남
- 튜닝과정
  - GC 상태 모니터링 하기
  - 알맞은 GC방식과 메모리 크기 설정
  - 적용하기
  
