
# 동기화란?
다수의 프로세스 ,스레드에서 같은 공유 자원에 동시에 접근하는 경우가 생깁니다.

이럴 경우 순서가 정해지지 않으면 데이터 일관성이 깨질 수 있습니다.

동기화란 다수의 프로세스 혹은 스레드 들이 공유 자원에 동시에 접근할 때 접근 순서를 보장해주는 것입니다.

# Race Condition

다수의 프로세스와 스레드가 공유자원에 동시에 접근하려는 상황을 말합니다.

공유 자원에 동시 접근은 데이터의 불일치 문제를 발생시킬 수 있습니다 따라서 Race Condition 을 막고 일관성을 유지하기 위해서 다수의 프로세스 간의 실행순서를 정해주는 동기화가 필요합니다.

# Critical Section

2개 이상의 프로세스가 동시에 접근하면 안되는 공유 영역을 말합니다. 즉 Race condition 이 발생할 수 있는 영역을 말합니다.

다수의 프로세스가 critical Section 에 동시에 접근하지 못하고 데이터 정합성을 유지하며 안전하게 실행하기위해서 상호 배제  , 진행 , 유한 대기 조건을 지켜야합니다.

- 상호배제 : 한 프로세스가 critical section 에 진입했다면 다른 프로세스는 critical section 에 진입할 수 없다.
- 진행 : critical section 에 어떤 프로세스도 진입하지 않았다면 임계 구역에 진입하고하는 프로세스는 진입할 수 있어야한다.
- 유한 대기 : critical section 에 들어가고자하는 프로세스가 있다면 언젠가는 들어가야한다.

# 동기화 기법

- 뮤텍스락 : 뮤텍스 락은 동시에 접근하면 안되는 자원에 동시에 접근할 수 없도록 하는 동기화 기법입니다.
    - 모든 프로세스들이 공유하는 lock 변수가 있습니다.
    - lock 변수값이  false 인 경우 대기해야합니다.
    - lock 변수값이  true 인 경우 획득할 수 있고 critical section 영역에 진입할 수 있습니다.
        
        진입 후에는 lock 변수값을 flase 로 지정해야합니다.
        
    - critical section 영역에서 나오는 경우 lock 변수값을 true 로 지정해야합니다.
- 세마포어 : 뮤텍스 락과 비슷하지만 조금 더 일반화된 방식의 동기화 도구입니다.
    
    세마포어는 공유 자원이 여러 개 있는 상황에서도 적용할 수 있는 동기화 도구입니다.
    
    - 모든 프로세스들이 공유 하는 전역 변수 S 를 공유 자원수로 초기화합니다.
    - 전역 변수 S 가 양수인 경우 critical section 에 진입할 수 있고 S 의 값을 감소시킵니다.
    - 전역 변수 S 가 음수인 경우 대기합니다.
    - critical section 에서 나오는 경우 S 를 증가시킵니다.
