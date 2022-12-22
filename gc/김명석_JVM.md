## JDK (Java Development Kit)
>Java 개발을 위한 JRE, Compiler, Debuger 등 개발도구

![image](https://user-images.githubusercontent.com/9216335/208895696-55bffdac-7616-4d0d-9654-2ac8a64c3fef.png)

## JRE (Java Runtime Environment)
>Java 프로그램 실행을 위한 런타임 환경

![image](https://user-images.githubusercontent.com/9216335/208896570-9f19eba2-51ec-4e04-9ca5-9b71202f8c2c.png)

## JVM (Java Virtual Machine)
>자바는 WORA(Write Once Run Anywhere)를 구현하기 위해 가상머신 기반으로 동작
- 스택 기반 모델로 많은 메모리가 필요
- 심볼릭 레퍼런스
	- 기본타입을 제외한 모든 타입이 메모리 주소기반 레퍼런스가 아닌 심볼릭 레퍼런스로 참조
- GC (Garbage Collection)
	- 클래스 인스턴는 사용자 코드에의해 생성되고 GC에 의해 파괴
- 네트워크 바이트오더 (빅 엔디안)

>[!note] Symbolic Reference
>참조하는 클래스의 메모리주소가 아닌 대상의 이름으로 참조 하는것
>![image](https://user-images.githubusercontent.com/9216335/208907692-def3ddec-4451-4027-93d7-ecba753098e2.png)
>![image](https://user-images.githubusercontent.com/9216335/208907755-ca574d99-ce12-4b00-b323-16d47ceb9432.png)

>[! note] Byte Order
>각각에서 0x12345678 이라는 정수가 어떤 식으로 저장되는지 살펴보자
>**Big Endian**
>낮은 주소에서 높은 바이트 부터 저장
>![image](https://user-images.githubusercontent.com/9216335/208900796-33f17885-a5a3-4aa1-b6e3-4a44d0920c75.png)
>**Little Endian**
>낮은 주소에서 낮은 바이트 부터 저장
>![image](https://user-images.githubusercontent.com/9216335/208902397-a94823c4-81a4-4144-b217-ca1aea076fde.png)

### JVM 구성 요소
![image](https://user-images.githubusercontent.com/9216335/208897070-89f9b89a-0af5-4456-9a90-b38d05d048a6.png)
- Class Loader
- Runtime Data Area
	- 모든 쓰레드가 공유하는 영역
		- Heap
		- Method Area
	- 각 쓰레드 마다 고유하게 생성되는 영역
		- Stack
		- PC Register
		- Native Method Stack
- Execution Engine
	- Interpreter
	- JIT(Just In Time) Compiler
	- GC(Garbage Collector)

### Class Loader
>.class 파일 (bytecode) 가 `Loading` - `Linking` - `Initializing` 을 거쳐 JVM 에서 사용됨

![image](https://user-images.githubusercontent.com/9216335/208879803-aeb53302-3a93-4a58-94ca-9a1e1f26ebde.png)  
- 모든 Class 들은 참조되는 순간 동적으로 Load 및 Link 가 이루어진다
- Class Loader 는 클래스를 동적으로 메모리에 로딩할 책임을 가진다

### Runtime Data Area
>실행 중인 프로그램의 정보가 올라가 있는 메모리
- Method Area : 클래스 구조를 메타데이터 처럼 가지고 있으며 메서드의 코드를 저장
- Heap : 어플리케이션 실행 중에 생성되는 객체 인스턴스를 저장
- Stack : 메서드 호출을 스택 프레임이라는 블록으로 쌓으며 로컬 변수, 중간 연산 결과들이 저장
- PC Register : 쓰레드가 현재 실행할 Stack Frame의 주소를 저장
- Native Method Stack : C/C++ 등의 Low level 코드를 실행하는 스택

### Execution Engine
>Java -> java bytecode(\*.class) -> binarycode로 변환
- Interpreter
- JIT compiler
- Garbage Collector
- **실행 과정**
	- Java Compiler (javac.exe) 가 .java 를 .class (bytecode) 로 변환
	- JIT(Just In Time) Compiler 가 코드 전체를 살펴 중복을 미리 체크
	- bytecode를 Interpreter로 실행하며 체크한 부분을 만나면 캐싱
	- 캐싱된 중복 코드를 다시 만나면 메모리에서 꺼내 실행

## DVM (Dalvik Virtual Machine)
> Android 에서 사용하는 가상머신

![image](https://user-images.githubusercontent.com/9216335/208885812-ff2b6505-172a-475c-9651-f7e0d16969b6.png)
- 레지스터 기반 모델로 적은 메모리에 최적화
	- 명령어가 단순하고 처리속도가 빠름
	- [스택기반모델과 레지스터 기반모델의 차이점](https://s2choco.tistory.com/13)
- 라이센스 문제를 피하기위해 만든 가상머신
- bytecode를 레지스터 기반 명령어코드(dex)로 변환
- 여러 VM 인스턴스를 실행
- Android 2.2 (API 8) 부터 Trace JIT 도입

>[!note] Trace JIT
>실행 시 자주 사용되는 부분의 바이트 코드를 기계어로 미리 해석
>translation cache 에 저장하면서 컴파일

## ART (Android Run Time)
> DVM 성능을 향상하기 위해 나온 방식 
- AOT 방식을 이용하여 앱 설치 시 컴파일을 수행하여 Runtime 성능을 향상시킨다
- VM 없이 실행 가능하지만 Dalvik과의 하위 호환성을 위해 VM위에서 돌아가는것 처럼 실행
- Android 4.4 (API 19) 에서 처음 등장 
- Android 5.0 (API 21) 에서 기본값이 됨
- Android 7.0 (API 24) 이후 AOT + JIT 방식으로 실행 

>[!note] AOT (Ahead Of Time) 란?
>- 앱 설치 시 컴파일
>- JIT 에 비해 설치 속도 느림
>- JIT 에 비해 실행 속도 빠름
>- 용량이 크다

### DVM vs ART
![image](https://user-images.githubusercontent.com/9216335/208912047-c5ca228a-6adf-4ff9-be61-51884ef56973.png)

### 🤔 ART 의 문제점
![image](https://user-images.githubusercontent.com/9216335/208889307-bbc117ce-a751-4812-b865-eb718c226dc8.png)
- 보안팀이 수정사항을 보내게 되면 시스템 업데이트가 발생
- 시스템 업데이트 시 새로운 시스템을 가지게 되므로 하드 코딩된 종속성이 모두 모효화 ART 가 새롭게 Binary 파일을 만들어야 한다
  -> Nougat 에서 JIT 를 다시 가져온 이유

### 🎊AOT + JIT 의 장점
- 시스템 업데이트 Dialog 의 제거
- 앱 설치 속도 75% 향상
- 실행할 코드만 컴파일 하여 저장공간 절약
- 베터리 소모량 감소

### N 에서 JIT를 다시 가져오면서 바뀐 사항은?
- M 에 비해서 더 빠른 Interpreter 를 구현
- UI 스레드가 아닌 별도의 스레드에서 JIT Compile 을 수행
- Method가 Compile 이후 더 이상 사용되지 않으면 메모리에서 제거

### 면접질문
- Android에서 JVM이 아닌 DVM 을 사용하게된 이유가 무엇인가?
- DalVik 과 ART 의 차이점은 무엇이가?
- ART의 단점이 무엇인가?
- android M 에서 Java 11 을 사용하면 어떤 GC 알고리즘이 적용되는가?
- android O 이상에서 적용되는 GC 알고리즘에 대해서 아는가?

### 참고자료
- [옵시디언 사용법](https://help.obsidian.md/How+to/Use+callouts)
- [DVM 에대한 고찰 - JDK, JRE, JVM](https://shrimp-burger.tistory.com/221)
- [JVM 이해하기 - 1](https://happy-coding-day.tistory.com/entry/JVM-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-1)
- [JVM Internal](https://d2.naver.com/helloworld/1230)
- [JIT Complier](https://kotlinworld.com/307)
- [JIT Compiler smackdounnkol Youtube](https://www.youtube.com/watch?v=8y0L9QT7U74&ab_channel=smackdounnkol)
- [Class Loader에 대해](https://nesoy.github.io/articles/2020-11/ClassLoader)
- [Google Docs ART](https://source.android.com/docs/core/runtime)
- [JVM DVM ART](https://velog.io/@sery270/JVM-DVM-ART)
- [Google I/O 2016 ART](https://www.youtube.com/watch?v=fwMM6g7wpQ8&ab_channel=AndroidDevelopers)
- [Google I/O 2018 ART](https://www.youtube.com/watch?v=vU7Rhcl9x5o&ab_channel=AndroidDevelopers)
- [Google I/O 2019 ART](https://www.youtube.com/watch?v=1uLzSXWWfDg&ab_channel=AndroidDevelopers)