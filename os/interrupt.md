# 인터럽트란?

인터럽트는 컴퓨터에서 입출력 등 하드웨어나 소프트웨어 예외 상황 등 특정 이벤트가 발생했을 때 현재 실행 중인 프로세스나 작업을 중단하고 그 이벤트에 대한 처리를 수행하는 매커니즘을 말합니다.

# 인터럽트를 사용하는 이유

I/O 작업등 중요한 하드웨어 이벤트를 처리하기 위해 , 다중 작업 처리를 위해 , 예외 처리를 위해

# 인터럽트 종류

- 프로그램 : 산술 오버플로우 , 0 나누기 , 시스템 콜 , 허용되지 않은 메모리 영역 침범 등
- 타이머 : 프로세서 안에 있는 타이머에 의해 발생합니다. 프로세서가 특정한 기능을 주기적으로 수행하도록 합니다.
- 입/출력 : I/O 컨트롤러에 의해 발생하며 주로 입출력 동작의 완료 , 프로세서로부터 요청을 받을 때 발생합니다.
- 하드웨어 failure : 전원 , 메모리 parity 등

# 하드웨어 인터럽트

- 전원 공급 오류
- CPU 또는 기타 하드웨어 장치 오류
- 타이머 인터럽트
- I/O

# 소프트웨어 인터럽트

- 시스템 콜
- 0으로 나누기
- 허용되지 않은 메모리 영역 침범
- 오버플로우

# 인터럽트 처리 과정

1. 인터럽트 발생
2. 수행 중이던 프로세스 정보를 PCB 에 저장
3. 인터럽트 서비스 루틴 주소를 찾음
4. 인터럽트 서비스 루틴을 통해 인터럽트 처리
5. 수행 중이던 프로세스 정보를 가져온 후 다시 수행
