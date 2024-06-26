# 프로세스와 스레드의 차이에 대해
프로세스는 뭐다.. 스레드는 뭐다..
# 꼬리 질문
## 프로세스 생명 주기
## 프로세스 간 통신 방법
## 스레드 간 데이터 공유 방법
## 멀티프로세싱, 멀티스레딩의 장단점
## 스레드 안정성
## 뮤텍스, 세마포어
## 컨텍스트 스위칭(프로, 스레 간)
## 스위칭 비용 줄이기
## 병렬성, 동시성
# 상세정리
## 프로세스와 스레드 기본 개념
- 프로세스
	- 운영체제에서 할당된 작업 단위
	- 메모리에 코드가 올라가 연산 등의 동작이 실행되어지고 있는 상태
	- 멀티 프로세스
		- 여러 개의 프로세스를 동시에 띄워두는 것
		- 프로세스 간에는 자원을 공유하지 않음
			- 중복되는 데이터가 메모리에 올라갈 수 있음
		- 프로세스 간 독립적 메모리 영역(스택,힙,데이터,텍스트) 생성
- 스레드 
	- 프로세스가 할당 받은 자원을 사용하는 실행 단위
	- 한 프로세스 내에서 분할되는 작업 흐름 단위
	- 여러 개가 존재 가능하며 자원 공유 이루어짐
## 개념 비교
### 자원구조
- 프로세스
	- 각 프로세스 당 독립적 메모리 영역(스택, 힙, 데이터, 텍스트) 생성
- 스레드
	- 한 프로세스 내에서 코드, 힙, 데이터 공유
		- 스택은 함수 실행에 필요한 값들 저장하는 메모리 공간
		- 스택이 독립되기에 각 스레드가 실행 흐름 자체를 독립적으로 가져갈 수 있음
### 자원 공유
- 프로세스
	- 기본적으로 각 프로세스는 독립적이나 다음의 방법 존재
	- IPC
		- Inter-Process Communication
		- 커널 단위로 제공하는 프로토콜
		- 종류
			- pipe("|") - 단방향 통신, 익명, 동시성 불가
			- named pipe - 양방향 통신, mkfifo, 동시성 불가
			- message queue
				- 메모리 공간.
				- 각종 미들웨어를 통해, 또는 직접 로직 구현
			- shared memory
				- 공유 메모리를 먼저 설정 후 프로세스 실행
			- socket
				- 네트워크 통신
				- 4계층의 프로토콜 사용 가능
				- 참고
					- 소켓은 그냥 나와 상대의 정보가 적힌 파일 이름
					- 웹소켓은 정확하게 같은 개념은 아님
			- LPC
				- Local inter-Process Communication
				- 이건 윈도우에서 제한적으로 사용되는 무언가 무언가임
				- 문서화도 안 됐담서 함부로 개념화된 이상한 넘임
- 스레드
	- 공유 자원을 함께 사용
## 병렬성, 동시성
- **병렬성**Parallelism
	- 명령을 실행하는 반도체 유닛을 물리적 코어 개수에 맞춰 병렬적으로 작업 동시 수행
	- cpu의 코어 개수, 스레드 개수로 성능 평가 가능
- **동시성**Concurrency
	- 동시에 실행되고 있는 척하는 것
	- 다량의 작업을 작게 세분화하고 빠르게 바꿔치기하며 수행하기에 사람 기준에서는 동시에 돌아가는 것으로 인식
	- 바꿔치기하는 작업이 컨텍스트 스위칭
- 병렬성을 두고 동시성을 사용하는 이유
	- 하드웨어적 한계
	- 논리적 효율
		- 오래 걸리는 작업이 끝날 때까지 다른 작업을 기다릴 수 없음
## 컨텍스트 스위칭
- 스케줄링 - 운영체제에서 CPU를 사용할 프로세스를 선택하고 할당
	- FCFS, SJF, Priority, Round-Robin, Multilevel Queue
- 컨텍스트 스위칭
	- 동시성에서 언급됐듯, 하나의 cpu가 다량의 작업을 번갈아 가면서 수행하는 것
	- os의 스케줄러는 프로세스들의 준비 실행을 번갈아가면서 이를 실현
- 프로세스 스케줄링
	- 5가지의 상태 지님
		- 생성, 준비, 실행, 대기, 종료
		- 기본적으로 생성 - 준비 - 실행 - 종료 순
		- 명령 로직(입출력, 이벤트)에 따라 실행 중 대기 상태 진입 경우 존재
		- 대기 끝날 시 다시 준비 상태로 복귀
		- 시그널, 예상되지 않은 이벤트를 통해 실행에서 준비로 가는 경우도 존재
	- 프로세스 컨텍스트 스위칭
		- Process Control Block
			- 프로세스 관리를 위해 캐싱하는 상태 정보
			- 메모리에 저장
			- 포인터, 상태, ID, 메모리 제한 등의 정보 포함
		- 과정
			- 한 프로세스 A 실행 중 인터럽트나, 시스템 호출 발생
			- A 상태 PCB 1에 저장 후 다음 실행할 프로세스 B 선택
			- PCB 2에서 B의 상태를 불러오고 실행
			- 한무 반복
		- 과정 상 상태 정보를 저장하고, 불러오는 스위칭 오버헤드 존재
			- PCB 저장 및 복원 비용
			- CPU 캐시 메모리 지우고 채우는 비용
			- 스케줄러 업무 비용
		- *스레드 간 스위칭에서도 크게 오버헤드 발생할 수 있음*
- 스레드 스케줄링
	- OS의 스케줄러와 같이 언어 별로 스레드 스케줄러 존재
	- Round-Robin, Preemptive-priority, FCFS, Time-slicing
	- 4가지의 상태
		- NEW, TERMINATED - 알잘딱깔센
		- RUNNABLE - 실행 기다리는 상태
		- BLOCKED  - 대기. 이벤트로 RUNNABLE되기 전까지 대기
	- 스레드 컨텍스트 스위칭
		- Thread Control Block
			- PCB에 담기는 스레드 저장 정보
			- 스레드 우선순위, 상태 정보 담김
		- 과정
			- 뮤텍스 - 임계구역에 하나의 스레드만 접근
			- 세마포어 - counter 개수만큼의 스레드 접근 가능
- 비교
	- TCB는 저장 정보가 가볍고 데이터 중복이 적어 훨씬 비용 적음
	- TCB는 메모리에 대한 캐시 메모리 삭제되지 않음
	- 그러나 TCB는 데이터를 공유해 실행 단위 간 데이터 침범 가능(경쟁조건)
## 장단점
- 프로세스
	- 독립적인 실행으로 직관적인 개발, 흐름 파악 가능
	- 중복되어도 괜찮은 영역에 대해서 메모리 낭비 많음
	- 컨텍스트 스위칭 비용 큼
### 스레드로 이득보기 어려운 상황
- 단일 프로세스로 단순 실행할 수 있으면 멀티 스레딩이 손해
	- 스위칭 오버헤드가 존재하기 때문
- 잔여 스레드가 남아 불필요한 시간 야기할 수 있음
	- 스레드 풀을 만들어 최대한 대응할 수 있음
- GUI 프로그램의 경우 싱글 스레드가 이득
	- 데드락, 순서 처리 이슈로 사용자 경험 해칠 가능성
- 소요가 많은 I/O 작업의 경우 이벤트기반 프로그래밍이 더 좋을 수 있음
	- Node.js - 싱글 스레드 + 이벤트 기반 프로그래밍
		- 사실 언어 자체가 싱글 스레드라고는 하지만, 결국 OS가 멀티 스레딩...
		- 대신 개발자는 동기화, 경쟁 상태 걱정 없이 프로그래밍 가능
## 멀티 프로세스 vs 멀티 스레딩
- 멀티 프로세스
	- 멀티 프로세스를 직접적으로 만들기 위해서는 시스템 콜 호출 필요
		- -> C언어 계열 개발자라면? SSAF가넝~
	- 이외엔 OS의 영역에서 일어나는 일이기에 일반적으로 개발자가 처리하지 않음
	- OS 멀티 프로세싱
		- 부모 프로세스로부터 하위 프로세스 생성
		- 서로의 정보는 가지지만 서로 독립적 공간
	- 장점
		- 안정성 -> 확장성
		- 병렬성을  잘 활용할 수 있음
		- 개발 쉬움
	- 단점
		- 컨텍스트 스위칭 비용 과다
			- cpu 내의 레지스터, 캐시 메모리 등이 전부 매번 초기화
		- 자원 공유 비효율
		- 문제 가능성
			- 좀비 프로세스 - 죽었는데 부모는 모르는 자식 프로세스
				- 종료 상태 잘 캐치하기
			- 고아 프로세스 - 부모가 먼저 종료된 자식 프로세스
				- 부모 종료 시 자식 프로세스 종료 잘 걸기
- 멀티 스레딩
	- 개발자가 직접 지정 가능
		- 언어별 차이 큰 편
		- C/C++
			- 직접적인 OS 스레딩 제어 가능
			- 저수준이기에 굉장히 빠름
		- Java
			- JVM 상 스레딩으로 경량 스레딩 가능
		- Python
			- GIL 문제로 사실 상 불가
			- I/O 등의 특정 작업에 대해서는 GIL이 해제되어 가능
			- 파?코루틴~
				- yeild, send 등으로 언어 단에서 스레딩과 비슷한 양상을 띄는 로직 가능
			- 모든 태스크가 힙 영역에서 관리되어 컨텍스트 스위칭 발생 X
		- 여기는 더 깊은 공부가 필요한 부분..
	- 장점
		- 데이터를 공유하며 가볍기에 훨씬 빠른 성능 보유
		- 컨텍스트 스위칭 비용 낮음
	- 단점
		- 동시 접근으로 인한 안정성 확보 어려움
			- 동기화 전략 필수
		- 동기화 전략으로 인한 문제
			- 성능 저하
				- 교착 상태(데드락) - 서로 임계영역 소유를 기다리다 막히는 상황
				- mutex, hold and wait, no preemtion, circular wait 등의 알고리즘
				- *멀티 프로세스도 공유 메모리 사용 시 같은 문제 가능성 존재*
			- 추가적인 개발 소요
		- 어려운 디버깅과 테스트
## 스레드 안정성
- 멀티 스레드 프로그래밍에서 동시 접근이 일어나도 이상 없는 상태
- 한 스레드가 어떤 상태이든 다른 스레드로부터 영향받지 않음
- 전략
	- **Mutual Exclusion**뮤텍스
		- 파이썬의 GIL
		- 동기화 전략 중 하나
	- **Atomic Operation**원자 연산
		- 자원 변경에 필요한 연산을 작은 단위로 분리하고 이에 대해서만 락 걸기
		- 함수형 프로그래밍의 대표적인 방식
	- **Thread-Local Storage**쓰레드 지역 저장소
		- 공유 자원 줄이고 각 스레드만의 저장소 확보
	- **Re-Entrancy**재진입성
		- 애초에 스레드 호출과 상관 없어서 언제 스레드끼리 스위칭이 일어나도 상관없게 만들기
- 참고
	- java
		- jvm에 책임을 전가하도록 syncronized를 걸지 않은 싱글톤은 unsafe
		- 참조는 unsafe
# 참고
- 인파냥 블로그
	- https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%E2%9A%94%EF%B8%8F-%EC%93%B0%EB%A0%88%EB%93%9C-%EC%B0%A8%EC%9D%B4
	- https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-Is-more-threads-always-better
	- https://inpa.tistory.com/entry/%F0%9F%91%A9%E2%80%8D%F0%9F%92%BB-multi-process-multi-thread
- 고아, 좀비
	- https://codetravel.tistory.com/31
- 파이썬 코루틴
	- https://velog.io/@jaebig/python-%EB%8F%99%EC%8B%9C%EC%84%B1-%EA%B4%80%EB%A6%AC-3-%EC%BD%94%EB%A3%A8%ED%8B%B4Coroutine
- 스레드 안정성
	- https://developer-ellen.tistory.com/205
	- https://parkcheolu.tistory.com/13
	- https://developer-ellen.tistory.com/205
- 프로세스 공유
	- https://jwprogramming.tistory.com/54
	- https://velog.io/@coastby/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-IPC-Inter-Process-Communication-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EA%B0%84-%ED%86%B5%EC%8B%A0#-shared-memory
	- https://velog.io/@coastby/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C-IPC-Inter-Process-Communication-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EA%B0%84-%ED%86%B5%EC%8B%A0#%E2%97%8B-socket
	- lpc
		- https://ko.wikipedia.org/wiki/%EB%A1%9C%EC%BB%AC_%ED%94%84%EB%A1%9C%EC%8B%9C%EC%A0%80_%ED%98%B8%EC%B6%9C
