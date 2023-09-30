## DBCP 란?

미리 정해진 개수의 커넥션을 만들어 Pool 에 저장하고 쿼리 요청에 따라 Pool 에 미리 생성해두었던 커넥션을 사용하고 요청이 끝나면 다시 커넥션을 Pool에 저장하는 방식을 말합니다.

## DBCP 를 사용하는 이유

매번 쿼리 전달을 위해 드라이버를 로드하고 커넥션 객체를 생성하여 매번 연결, 종료를 수행하기에는 비효율적이다. 뿐만 아니라 요청이 한순간에 몰리게되면 너무 많은 커넥션을 DB에서 생성하고 연결하게 되어 메모리 문제와 컨텍스트 스위치 같은 오베허드가 발생할 수 있다.

이러한 문제를 해결하기 위해 커넥션풀을 사용하여 커넥션 생성 및 종료 비용을 줄이고 너무 많은 커넥션이 생성되는 상황을 방지할 수 있다.

## 커넥션의 개수

### DB 서버에서 커넥션 개수

- `max_connections` : client 와 맺을 수 있는 최대 Connection 수
- `wait_timeout` : connection 이 inactive 할 때 다시 요청이 오기까지 얼마의 시간을 기다린 뒤에 close 할 것인지를 결정하는 시간 , 비정상 연결의 경우 커넥션을 종료하기 위해 필요함.

### Backend 서버에서 DBCP 설정

- `minimumidle` : Pool 에서 유지하는 최소한의 idle 상태의 connection 수
- `maxmumPoolSize` : Pool 이 가질 수 있는 최대 Connection 수 (idle + in-use)

idle 상태의 커넥션 수가 `minimumidle` 보다 작다면 `minimumidle` 수 까지 커넥션을 생성하는데  `maxmumPoolSize` 까지만 생성한다.

이 둘의 값이 같은 경우를 Pool 사이즈가 고정임을 의미한다. 요청이 늘어날 때 동적으로 커넥션을 생성하고 종료하는 것보다 Pool 사이즈를 고정하는 것이 성능적으로 나으며 이를 권장한다.

- `maxLifeTime` : Pool 에서  connection 의 최대 수명을 의미 , DB 의 Connection time limit 보다 짧게 설정해야함. 왜냐하면 Pool 에서 Connection 을 `maxLifeTime` 직전에 사용하고 쿼리를 전달하기 전에 DB 서버에서 `wait_timeout` 으로 인해 커넥션을 종료하게 된다면 정상적으로 처리되지 않을 가능성이 있기 때문이다. 따라서 maxLifeTime + 쿼리 전달 시간 < `wait_timeout` 이어야한다.
- `connectionTimeout` : Pool 에서 connection 을 받기 위한 대기 시간

## 적절한 Connection 수를 찾기 위한 과정

1. 모니터링 환경 구축 (서버 리소스 , 서버 스레드 수 , DBCP )
2. 백엔드 시스템 부하 테스트 (ngrinder , Jmeter)
3. `request per second` , `avg response time` 을 확인
4. 백엔드 서버 , CPU , MEM 등 리소스 사용률이 한계에 도달하는 경우 서버를 늘려서 대응
5. DB 서버의 부하가 늘어나는 경우 secondary 추가 , Cache layer 추가 , sharding 등으로 대응
6. 리소스등의 상태가 안정적인데 TPS 가 늘어나지 않는 경우에는 스레드 수를 확인하고 병목 확인하고 대응
7. DBCP 의 active connection 수를 확인하고 대응

(단 , backend 서버의 maximumPoolSize 수들의 합보다 DB 커넥션 수가 크거나 같아야한다.)
    

# Reference

---

- https://d2.naver.com/helloworld/5102792
- https://www.youtube.com/watch?v=zowzVqx3MQ4&t=91s
- https://hudi.blog/dbcp-and-hikaricp/
- https://zzang9ha.tistory.com/376
- https://perfectacle.github.io/2022/09/25/hikari-cp-time-config/#more
