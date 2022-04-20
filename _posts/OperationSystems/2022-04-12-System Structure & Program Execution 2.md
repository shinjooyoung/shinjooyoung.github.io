---
title:  "System Structure & Program Execution 1"
excerpt: "운영체제 - 반효경"
categories:
  - Study
tag:
  - ObjectOrientation
toc: true
---

## 동기식 입출력과 비동기식 입출력

### 동기식 입출력(synchronous I/O)
- I/O 요청 후 입출력 작업이 완료된 후에야 제어가 사용자 프로그램으로 넘어감
- 구현방법 1
  * I/O가 끝날 때까지 CPU를 낭비시킴
  * 매시점 하나의 I/O만 일어날 수 있음
- 구현방법 2
  * I/O가 완료될 때까지 해당 프로그램에게서 CPU를 빼앗음
  * I/O 처리를 기다리는 줄에 그 프로그램을 줄 세움
  * 다른 프로그램에게 CPU를 줌

### 비동기식 입출력(Asynchronous I/O)
- I/O가 시작된 후 입출력 작업이 끝나기를 기다리지 않고 제어가 사용자 프로그램에 즉시 넘어감

> 두 경우 I/O의 완료는 인터럽트로 알려줌

### DMA(Direct Memory Access)
- 빠른 입출력 장치를 메모리에 가까운 속도로 처리하기 위해 사용
- CPU의 중재 없이 device controller가 device의 buffer storage의 내용을 메모리에 block 단위로 직접 전송
- 바이트 단위가 아니라 block 단위로 인터럽트를 발생시킴

### 저장장치 계층 구조

Registers > Cache Memory > Main Memory > Magnetic Disk > Optical Disk > Magnetic Tape

- Cpu가 직접 접근에서 처리할 수 있는 곳은 Primary(Registers ~ Main Memory)
- Secondary는 바로 접근이 되지 않는다다(Magnetic Disk)

## 프로그램의 실행(메모리 load)
프로그램이 컴퓨터에서 실행될때는 파일 시스템에 실행 파일 형태로 저장되어 있다.  
이런 실행파일들을 실행하면 메모리에 올라가 프로세스가 되는데 바로 올라가는게 아닌 Virtual memory를 통해 올라가게 된다.

### Virtual memory

- 프로그램을 시작하면 프로세스에 대해 Address space 메모리 주소 공간이 생긴다.
  * 독자적으로 구성되며 stack(함수등의 데이터가 쌓이는 곳), data(변수, 전역변수, 자료구조), code(기계어 코드)로 구성된다. 
- 이런 독자적인 공간을 실제 메모리에 올라가서 실행되는데 프로그램이 종료되면 주소공간은 사라진다.
- 주소 공간이 모두 올라가는게 아닌 실행하는 코드만 올라가서 메모리 낭비가 되지 않는다.
- 결론적으로 가상 메모리의 값들은 Physical memory, Swap area(하드 디스크)에 저장된다.
  * Swap은 메모리 용량의 한계로 memory의 확장개념으로 사용된다.

**커널 주소 공간의 내용**
- 커널도 하나의 프로그램이기 때문에 code, data, stack이 구성된다.
- code
  * 시스템콜, 인터럽트 처리 코드
  * 자원 관리를 위한 코드
  * 편리한 서비스 제공을 위한 코드
- data
  * 운영체제가 사용하는 자료 구조
  * CPU, MEM, DISK 하드웨어를 관리해야 하는 자료구조를 하나씩 만들어 관리한다.
  * 각 프로그램들(Process A, Process B ...)을 관리하기 위한 자료구조가 구성되어 있다.(PCB)
- stack
  * 함수 호출이나 리턴될때 사용된다.
  * 사용자 프로그램마다 운영체제 커널 시스템을 호출해서 실행되기 때문에 프로그램 마다 별도의 커널 스텍을 따로 두고있다.


### 함수
- 사용자 정의 함수
  * 자신의 프로그램에서 정의한 함수
  * 직접 작성한 함수를 불러서 사용함
- 라이브러리 함수
  * 자신의 프로그램에서 정의하지 않고 갖다 쓴 함수
  *자신의 프로그램의 실행 파일에 포함되어 있다.
- 커널 함수
  * 운영체제 프로그램의함수
  * 커널 함수의 호출 == 시스템 콜

> 사용자 정의함수, 라이브러리 함수는 프로세의 Address space 영영으로 각 시스템에서 점프에서 실행되고 
> 커널함수는 Kernel Address space로 구성되고 커널에서 실행(시스템 콜을 통해 인터럽트 라인을 구성)이 된다. 


