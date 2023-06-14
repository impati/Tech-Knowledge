# 프로세스 스케쥴링


프로세스 스케쥴리은 주어진 프로세스가 여러 개인 경우 어떤 순서대로 프로세스를 처리할지 운영체제가 결정하는 것을 말합니다.

# 스케쥴링 Criteria


여러 스케쥴링 알고리즘 중 각 알고리즘의 성능을 평가하는 기준을 말합니다.

- 시스템 입장에서의 성능 척도 :
    - CPU 이용률  : 전체 시간 중 CPU 가 수행되는 비율
    - 처리량 : 단위시간당 처리하는 프로세스의 수
- 프로세스 입장에서의 성능 척도 :
    - 소요 시간 : Ready Queue 에서 대기한 시간부터 작업을 완료할 때까지 걸리는 시간
    - 대기시간 : CPU 를 점유하기 위해  Ready Queue 에서 기다린 시간
    - 응답 시간 : 프로세스가 처음으로 CPU 를 할당받기까지 걸린 시간

# 스케쥴링 정책

---

- 선점 스케쥴링 정책
    - 선점 스케쥴링 정책은 실행 중인 프로세스에 인터럽트를 걸고 다른 프로세스를 CPU에 할당할 수 있는 스케쥴링 방식입니다
    - 선점 방식은 높은 우선순위를 가진 프로세스를 우선적으로 처리해야하는 경우 유리합니다.하지만 우선순위가 높은 프로세스만이 실행될 수 있는 문제가 발생할 수 있으며 자주 선점이 발생한다면 문맥 교환에 따라 성능이 저하될 수 있습니다.
- 비선점 스케쥴링 정책
    - 실행 중인 프로세스를 바로 준비 상태로 전이 시킬 수 없는 방식으로 CPU 를 점유한 프로세스는 종료되거나 대기 전까지 계속 실행할 수 있습니다.
    

# 스케쥴링 알고리즘


## 비선점 스케쥴링 알고리즘

- First Come First Served 스케쥴링
    - 비선점 스케쥴링 방식입니다.
    - 먼저 Ready Queue 에 온 프로세스가 CPU를 점유하는 방식입니다.
    - 먼저 실행된 프로세스의 수행시간이 긴 경우 나중에 실행될 프로세스가 오래 기다릴 수 있습니다.이를 **Convoy Effect** 라고합니다.

- Shortest Job FIrst 스케쥴링
    - 비선점 스케쥴링 방식입니다.
    - **Convoy Effect** 를 해결하기위한 방식으로 프로세스의 수행 시간이 짧은 순서대로 CPU 를 점유하는 방식입니다.
    - Shortest Job FIrst 스케쥴링은 항상 주어진 프로세스에 대해 최소의 평균 대기 시간을 보장합니다.
    - 프로세스가 얼마나 CPU 를 사용할지 예측하기 어렵다는 단점이 있습니다.

비선점 스케쥴링 방식이므로 실시간으로 실행되어야하는 시스템에서는 적합하지 않습니다.

## 선점 스케쥴링 알고리즘

- Shortest Remaining Time First 스케쥴링
    - 선점 스케쥴링 방식입니다.
    - 현재 준비큐 프로세스 중 남은 시간이 가장 짧은 프로세스를 실행하는 알고리즘입니다.
    - 남아있는 실행 시간을 알고 있어야한다는 단점을 가지고 있습니다.
    - 남아 있는 실행 시간이 매우 큰 프로세스의 경우 수행되지 못하는 Starvation 이 발생할 수 있습니다.
    
- Round Robin 스케쥴링
    - 선점 스케쥴링 방식입니다.
    - Ready Queue 의 여러 프로세스가 동일한 시간을 수행한뒤 다시 Ready Queue 에 돌아가는 방식입니다
    - Ready Queue의 모든 프로세스가 동일한 시간동안 수행하도록 순서대로 CPU 를 점유하므로 Starvation 이 발생하지 않습니다  .
    - 프로세스가 실행되는 시간이 작을 수록 문맥교환으로 발생하는 오버헤드가 커질 수 있습니다.
    
- Priority 스케쥴링
    - 특정 기준으로 프로세스에게 우선순위를 부여해 우선순위가 가장 높은 프로세스가 CPU를 점유하는 방식입니다.(일반적으로 숫자가 작을수록 높은 우선순위를 가지고 있음을 의미합니다.)
    - 우선순위가 낮은 프로세스의 경우 Ready Queue 에 항상 높은 우선순위를 가진 프로세스가 있는 경우
        
        Starvation 이 발생할 수 있습니다.
        
    - Aging 기법은 오래기다린 프로세스의 우선순위를 높혀 Startvation 문제를 해결할 수 있습니다.
    

## MultiLevel Queue

멀티 레벨 큐는 Ready 큐를 여러개로 분할 것입니다. 프로세스의 성격에 따라 프로세스 그룹으로 나눈뒤 우선순위에 맞게 스케쥴링 하는 방법입니다.

- System Processes : 커널수준 프로세스
- Interactive Processes : 대화형 프로세스
- Batch Processes : 일정량을 한번에 처리하는 프로세스

System Processes가 높은 우선순위를 가지고 Batch Processes 가 낮은 우선순위를 가집니다.

높은 우선순위를 가질 수록 CPU 를 점유하는 빈도는 많지만 시간이 적으며 Round Robin 방식으로 동작합니다.

반면에 낮은 우선순위를 가질 수록 CPU 를 점유하는 빈도는 적지만 시간이 많습니다. FCFS 방식으로 동작합니다.

## MultiLevel Feedback Queue

MultiLevel Queue 에서 프로세스가 여러 큐들 사이에서 이동할 수 있는 성질이 추가된 것입니다.

높은 우선순위에서 적은 시간동안 CPU 를 할당받고 실행합니다.

작업이 완료되지 못한 경우에는 점점 낮은 우선순위 큐에서 많은 시간 동안 CPU 를 할당받아 실행하며 동작합니다.