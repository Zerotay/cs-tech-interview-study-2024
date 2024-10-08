# 파드에 대해서
쿠버네티스는 내부에 존재하는 모든 것들을 객체라고 부른다.
이를 통해 쿠버네티스에서 설정하고 싶은 것들, 네트워크나 볼륨 등의 요소를 다룬다.
쿠버네티스의 가장 기본이 되는 객체는 파드(pod)라고 부른다.
오늘은 파드의 기본 개념과 사용법을 익히며 감을 잡는 시간을 가진다.
기본이 되는 만큼, 알아야 할 개념이 많다.
# 개념
> 파드는 쿠버네티스에서 만들고 관리할 수 있는 최소 컴퓨팅 배포 단위이다.

![[Pasted image 20240810102053.png]]
pea pod가 여러 개의 완두콩을 담고 있듯이.

파드는 쿠버네티스에서 가장 작은 관리 단위이다.
파드는 여러 개의 컨테이너로 이뤄져있어서 기본적으로 컨테이너보다는 큰 개념이다.
내부의 컨테이너들은 전부 스토리지와 네트워크 자원을 공유한다.
이들은 하나의 컨텍스트 안에서 위치하고, 공동 관리가 이뤄진다.

가장 직관적일 수 있을 예시는 도커 컴포즈라고 생각한다.
도커 컴포즈 역시 여러 개의 컨테이너를 서비스로 정의하고, 볼륨과 네트워크를 공유하도록 편하게 관리할 수 있다.
완벽히 같지는 않지만(네트워크 방식, 개별 실행 여부 등), 실제 사용 모습도 꽤나 비슷해 보이는 편이다.

## 기본 양식
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```
기본적인 파드를 만드는 방법.
그러나 실질적으로 이렇게  기초적인 형태로 만들기보다는 replica나 다른 워크로드의 관리 대상으로서 만들어지는 게 더 일반적이다.
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```
이건 deployment manifest의 예시
```yml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```
이건 job의 예시
이런 식으로, template 내부에 들어가는 경우가 대부분이다.
통상 여러 개의 파드를 스케일아웃하는 식으로 관리한다. 이를 통해 파드의 동작이 실패하거나 개수 조정이 필요할 때 파드가 조작된다. 
나중에 하나씩 보게 될 텐데, 다음의 리소스들이 파드를 관리한다.
- [[Deployment]]
- [[StatefulSet]]
- [[DaemonSet]]
- [[Job]]

참고로 이렇게 파드를 관리하는 리소스들에서 파드에 대한 정보를 바꿔서 다시 적용한다면, 기존에 있던 파드가 변형되지는 않는다.
기존의 파드와 다른 새로운 파드가 띄워지고, 기존의 파드가 종료되는 식으로 동작한다.
## 파드와 컨테이너
> [!NOTE] 왜 파드가 기본 단위일까?
> 왜 컨테이너가 기본 단위가 아닐까?
> 컨테이너는 기본적으로 휘발적이며, 프로세스가 죽으면 자동으로 회복하는 등의 다양한 기능을 포함하지 않는 단순 격리된 프로세스이다.
> 쿠버네티스는 이러한 컨테이너들을 관리하며 자동 회복, 스케일링 등의 작업을 수행하기 위해 존재하는 툴이기에 이를 감싸는 파드를 기본으로 두는 것이라고 생각해볼 수 있다. 

컨테이너는 하나의 격리된 프로세스이다.
이것을 감싸고 관리하는 환경으로서 파드가 존재한다.
유의할 점은 컨테이너가 종료된다고 파드가 종료되는 것이 아니라는 것이다.
마찬가지로 컨테이너가 재시작된다고 해도 파드는 기본적으로 계속 지속된다.
파드가 사라지는 경우는 다음과 같다.
- 작업이 완료되었을 때(지정된 작업이 명시된 경우)
- 파드 객체가 제거(delete)되었을 때
- 리소스 부족으로 퇴출(evict)되었을 때
- 노드 호출이 실패했을 때

파드는 쿠버네티스에서 수명 주기를 가지고 관리되고, 이 파드가 내부 컨테이너의 수명 주기를 관리한다고 보면 편할 것 같다.
(이건 명확한 표현은 아니지만, 개략적인 이해에는 도움된다고 생각한다.)

파드의 기본적인 사용 방법은 하나의 컨테이너를 두는 것이다.
하지만 밀접하게 연관된 여러 개의 컨테이너가 하나의 기능으로 동작한다면 여러 개를 두는 것도 충분히 고려할 만하다.
나중에 보겠지만, 하나의 컨테이너를 둔다고 하더라도 기본적으로 파드는 사용자가 설정하지 않은 init container, pause container 등의 추가적인 컨테이너가 함께 설정되어 작동한다.

## 공유 자원
파드 내에서 공유되는 자원은 대표적으로 스토리지와, 네트워크가 있다.
스토리지에 대해서는 이후에 조금 더 자세히 다룰 예정.

네트워킹은 이후에 자세히 볼 것이지나, 조금 알아둬야 할 부분이 있다.
각 파드는 클러스터 내부에서 고유한 ip를 가진다.
그리고 파드 내 컨테이너들은 이 ip를 공유하게 된다.
그렇기에 서로 같은 포트를 사용할 수 없고, 대신 `localhost`로 서로를 호출할 수 있다.
뿐만 아니라 [[IPC]]와 같은 프로세스 간 통신 방법도 사용할 수 있다.
파드 끼리의 통신을 하고자 한다면 파드를 다른 리소스에 노출시키는 객체, [[Service]]를 사용해야 한다.
# 구조
이 부분에서는 파드와 관련된 각종 개념들에 대한 심화를 다룬다.
## 파드 종류
사실 종류라고 해봐야 문서에서 정의되는 다른 파드는 하나밖에 없는 것 같긴 하지만..
## static pod
파드 중에는 특별한 파드가 있다.
이름하야 스태틱 파드.
기본적으로 파드는 우리가 적용하기 위해 [[kube-apiserver]]에 상호작용해야만 한다.
그러나 각 노드들의 [[kubelet]]에 직접적으로 관리되는 파드들이 존재한다.
이들은 대체로 클러스터를 관리하는데 사용되는 파드들이다.
그리고 웬만해서 그러면 안 되겠지만, 사용자가 직접 수정을 가하면 해당 사항이 즉각 반영된다.

`kube-system`에 위치한 파드들이 바로 스태틱 파드들인데, 이들은 우리가 kubectl로 관리를 하려고 해도 제대로 반영되지 않는다.
이것들이 위치한 디렉토리가 존재하는데, 여기에서 수정을 가하면 즉각 반영된다.
이것도 이후에 심화로 다룰 예정.
## 컨테이너 종류

### 앱 컨테이너
우리가 직접 만들어서 관리하게 될 컨테이너.
### init container
### sidecar container
### pause container
## 프로브
컨테이너에 대해 [[kubelet]]이 주기적으로 수행하는 진단을 프로브라고 부른다.
이를 통해 컨테이너의 상태를 확인하고, 이로 말미암아 파드의 상태를 정의내릴 수 있다.
가령 livenessprobe를 설정해서 어떤 컨테이너의 헬스체크가 실패하면 파드가 실패했다고 정의를 내리는 것이 가능하다.
이렇게 되면 파드가 실패되었다고 간주되므로 파드는 자동으로 재시작 작업을 하게 된다.

# 라이프사이클
파드의 생애주기를 알아보자.
파드는 먼저 스케줄링된 후에 실행된다.
## 스케줄링
파드를 만드는 것에 대한 요청이 들어오는 것이 가장 먼저 선행된다.
구체적으로는 etcd에 파드에 대한 정보가 기록되고, 이를 특정 노드의 kubelet이 인지하는 과정이다.
이 과정은 전체 주기에서 한번만 발생한다.
이를 조금 더 세분화하면 이런 과정을 거친다.
- 스케줄링
	- [[kube-scheduler]]가 어떤 노드에 파드가 들어가야 할지 지정하는 과정
	- 적절한 정책에 따라 들어갈 만한 위치의 노드를 특정하면 이를 [[etcd]]에 정보로 기록한다.
- 바인딩
	- 파드를 특정한 노드에 할당하는 과정
	- 스케줄링이 된 이후, kubelet이 etcd의 정보를 읽어들인 후에 자신 노드에 파드가 생성돼야 한다는 것을 인지하는 과정이다.
	- 이후 kubelet은 본격적으로 파드를 생성하는 동작을 실행한다.

과정을 이렇게 세분화하는 이유는 다음의 상황이 존재하기 때문이다.
가령 어떤 노드가 모종의 이유로 파괴되거나 연결이 끊긴다고 생각해보자.
- 파드가 스케줄링되기 이전
	- 한 노드가 사용할 수 없는 상태이기에 스케줄러는 다른 노드에 파드를 스케줄링한다.
- 파드가 해당 노드에 스케줄링된 후
	- 이미 스케줄링은 완료되었다는 것은 이미 etcd에 정보가 남았다는 뜻
	- 이미 해당 노드는 사용 불가이기에 해당 노드의 kubelet이 파드를 만드는 작업이 없다는 것이기도 하다.
	- 즉, 쿠버네티스 입장에서 해당 파드는 실행되지 않게 되는 것이다.
		- 이 상황에 대해서 쿠버네티스는 파드를 건강하지 않은 상태로 진단하고 파드가 죽었다고 상태를 정의한다.
		- 파드는 리소스 부족이나 노드의 파괴로부터 살아남지 못한다.(A Pod won't survive an [eviction](https://kubernetes.io/docs/concepts/scheduling-eviction/) due to a lack of resources or Node maintenance.)
		
참고로 1.30 버전부터 [[pod scheduling readiness]]라 하여 스케줄링 게이트가 제거될 때까지 파드 스케줄링을 연기하는 전략이 가능하다.

> [!NOTE] 파드의 회복
> 통상적으로 파드가 회복될 수 없는 상태라면 쿠버네티스는 새로운 파드를 만들어버리는 것을 선택한다.
> 이를 탐지하고 관리하는 작업은 [[kube-controller-manager]]가 담당한다.
> (새로운 파드를 어디에 할당할지는 또다시 [[kube-scheduler]]의 일이 될 것이다.)
> 반대로 말하면 이 경우 동일한 UID를 가진 파드를 유지할 수는 없다는 것이다.
> 그래도 이름은 똑같이 할 수 있다!
> 이를 주의 깊게 여겨야 하는 이유는 스토리지 문제에 있다.
> 새로 파드가 만들어진다는 것은 이전 파드가 사용하던 볼륨이 유지되지 않는다는 것이다.
> 따라서 이전 파드에 중요한 정보가 휘발되지 않도록 하는 대책이 요구된다.

## 파드 단계(phase)
바인딩이 된 이후의 파드는 다음의 상태를 가지게 된다.
이 단계는 매우 단순하고 추상 요약된 상태에 불과하다는 것을 유의해야 한다.
대신 이 상태 이외의 상태는 존재하지 않고, 또한 이 순서는 반드시 보장된다.
- Pending
	- 클러스터 내에 파드가 생성되었다.
	- 그러나 내부 컨테이너가 아직 실행할 준비가 안 된 상태이다.
	- 가령 컨테이너 이미지를 받고 있는 상황이라는 것.
- Running
	- 파드가 노드에 바인딩됐다.
	- 내부에 최소 하나의 컨테이너가 실행 중이거나 시작, 재시작 중이다.
		- running이라 해서 무조건 파드가 원하는 대로 움직이는 중이라는 것은 아닐 수도 있다는 것에 유의하자
- Succeeded / Failed
	- succeeded
		- 모든 컨테이너가 성공적으로 종료되었고, 재시작되지 않는다.
	- failed
		- 최소 하나의 컨테이너가 실패로 종료되었다.
		- 종료되면서 0이 아닌 값이 시스템에 반횐되었고 자동 재시작이 불가능한 상태이다.
			- 컨테이너가 재시작이 불가능한 상태는, 아래에서 더 자세히 보자.
	- 이 상태들은 컨테이너 종료 반환값을 토대로 결정된다.
- Unknown
	- 모종의 이유로 파드의 상태가 확인되지 않는다.
	- 이는 노드 통신이 불안정할 때 주로 발생한다.

참고로 파드가 제거되는 중에 `Terminating`이라는 상태가 출력된다.
그러나 이는 페이즈로 인정되지 않는다.
기본적으로 파드는 안전한 종료 텀으로 30초 동안 이 상태를 유지한다.
이를 깨고 싶다면 종료 조건을 양식에 작성하거나, `--force` 옵션을 주도록 한다.

보통 노드가 클러스터에서 사라진다면 해당 노드에 위치한 모든 파드는 failed 상태가 된다.
failed는 위에서 언급한 대로 재시작이 불가능한 상태를 말한다!
그렇지만 재시작 정책을 어떻게 설정하느냐에 따라 이를 해결할 수도 있다.
## 컨테이너 상태
파드의 상태와 컨테이너의 상태는 별개이다.
kubelet이 [[container runtime]]을 통해 파드를 만드는 과정 속에서, 컨테이너는 다음의 세 가지 상태를 가진다.
- Waiting
	- 다른 상태가 아니라면, 컨테이너는 기본적으로 이 상태이다.
	- 컨테이너는 running 상태가 되기 위해 필요한 각종 작업을 진행하고 있다.
	- 가령 컨테이너 이미지를 받고 있거나, [[Secret]] 데이터를 적용하고 있는 상태
- Running
	- 컨테이너가 이슈 없이 실행되고 있는 상태
	- `postStart` 훅은 이 상태 이전에 완료됨
- Terminated
	- 컨테이너가 성공적이든, 실패했든 종료 작업을 실행하는 상태
	- `preStop` 훅이 이 상태 이전에 작동

[[container lifecycle hooks]]를 통해 컨테이너 상태에 따른 각종 작업을 진행하거나 디버깅을 할 수 있다.
위의 postStart, preStop이 이 내용에 해당한다.
### 컨테이너 실패에 대하여
`restartPolicy`를 양식에 작성하여 컨테이너 실패에 대한 관리를 할 수 있다.
#### 재시작 절차
이 정책은 다음의 과정을 발생시킨다.
- initial crash
	- 처음으로 실패가 발생했을 때
	- 쿠버네티스는 `restartPolicy`에 정의된 정책을 토대로 동작을 시도한다.
- repeated crashes
	- 첫 충돌이 발생한 후 쿠버네티스는 역시 정의된 딜레이 시간에 맞춰 연속적으로 재시작을 시도하게 된다.
	- 이때 시간을 점진적으로 늘리면서 재시작 주기를 길게 늘린다.
	- 이를 통해 과도하게 재시작을 하는 상황을 막는다.
- CrashLoopBackOff state
	- 충돌과 재시작이 지속적으로 반복되고 있다는 것을 의미한다.
- Backoff reset
	- 컨테이너가 일정 시간동안 성공적으로 실행되었을 경우
	- 쿠버네티스는 재실행 연기 시간을 초기화하고, 이후에 발생한 실패를 첫 실패로 간주한다.
#### CrashLoopBackOff
이 중 `CrashLoopBackOff` 상태는 컨테이너 실패에 따라 자주 보게 되는 상태일 것이다.
크룹백은 언제 주로 발생하는가?
- 어플리케이션이 에러를 일으켜 컨테이너가 종료될 때
- 설정 파일이나 환경 변수가 없어서 실행 상의 에러 발생
- 메모리나 cpu가 충분하지 않아 리소스 제한이 걸렸을 때
- 예정된 시간 동안 어플리케이션이 헬스체크를 실패했을 때
- liveness probe에서 실패 상태가 적용될 때
- 등등..

뭐가 많지만, 결국 컨테이너가 제대로 실행되지 않는 상태들이다.
하지만 이 에러를 대응하기 위해서는 다양한 방법을 시도해볼 필요가 있어서 이렇게 세분화한 것이다.
얼마나 이 에러에 대한 질문이 많았으면 공식 문서에도 떡하니 박아놨다.
- 로깅
	- `kubectl logs`를 통해 로그를 확인한다.
- 이벤트 확인
	- `kubectl describe pod`를 통해 리소스나 설정에 관한 힌트를 얻는다.
- 설정 검토
	- 환경 변수나 볼륨 마운트, 외부 자원이 전부 제대로 설정되었는지 확인하기
- 리소스 제한 확인
	- 수동으로 리소스를 제한할 때 지나치게 적게 리소스를 부여한 것이 아닌지 체크해본다.
- 앱 디버깅
	- 앱 자체에서 발생하는 이슈일 수도 있으니 개발자더러 확인하라 시킨다.
	- 개발 환경과 배포 환경이 다른 데에서 이슈가 발생할 가능성도 존재한다.
#### 재시작 정책
그래서 재시작 정책에는 무엇이 있단 말인가?
- Always
	- 종료 후에는 항상 재시작
	- 정상 종료여도 무조건 재시작이나, init 컨테이너는 한번만 실행된다.
		- [[init container는 재시작되는가?]]
- OnFailure
	- 비정상 종료일 때만 재시작
- Never
	- 무조건 재시작하지 않음
	- 파드에서 failed 상태가 나는 이유 중 하나.
	- [[파드가 failed 뜨는 상황이란]]에 실험을 진행했다.
	- 이게 아니면 파드 자체가 failed 뜨는 일은 잘 없다.

이 정책은 파드 내부의 앱 컨테이너, 초기화 컨테이너에 대해 동작한다.
사이드카 컨테이너는 이를 무시한다.
사이드카는 재시작 정책이 always인 초기 컨테이너 내부에서 정의되기 때문이다.

재시작은 항상 같은 노드 내에서 이뤄지는 것이 보장된다.

# 설정 방법



# 참고
- https://kubernetes.io/docs/concepts/workloads/pods/
	- 와 하위 문서들까지
- 파드 스케줄링 게이트
	- https://kubernetes.io/docs/concepts/scheduling-eviction/pod-scheduling-readiness/
