# 가상화?

---

- 물리적인 하드웨어 자원을 논리적인 객체로 추상화하는 것을 의미합니다.
- 가상화를 하는 이유는 잉여 자원을 더욱 더 효율적으로 사용할 수 있게 되고 분산처리를 가능하게 할 수 있습니다.
- 가상화를 통해 사용자가 많은 서버에는 자원을 많이 할당해주고 적은 서버에는 자원을 적게 할당해주며 유연하게 대응이 가능해졌습니다

# 서버 가상화

---

- 하나의 물리적 서버 호스트에서 여러 개의 서버 운영체제를 게스트로 실행할 수 있게 해주는 소프트웨어 아키텍처로
    
    하이퍼 바이저라는 기술 통해 여러 개의 운영체제를 하나의 호스트 운영체제에 생성되고 실행됩니다.
    
- 하이퍼 바이저는 게스트 운영체제의 커널을 번역해서 하드웨어에 전달하는 역할을 수행합니다.
- 게스트 운영체제는 완전히 독립적인 공간과 시스템 자원을 할당 받아 사용합니다

# 가상화 단점

---

- 시스템 자원을 가상화하고 독립된 공간을 생성하는 작업은 하이퍼바이저를 거치기 때문에 일반 호스트에 비해 성능 손실이 발생합니다.
- 게스트 운영체제를 사용하기 위한 라이브러리 , 커널 등을 전부 포함하기 때문에 용량이 크다.

# 컨테이너 기반 가상화

---

- 독립적인 실행 환경을 가지는 어플리케이션을 실행하기 위해 무거운 게스트 OS 까지 필요없다.
- 컨테이너 기술을 프로세스 단위의 격리 환경을 만듭니다.
- 애플리케이션을 실행하기 위해 필요한 모든 소프트웨어와 종속성을 포함하는 격리된 환경입니다.
- 성능 손실도 게스트 OS 에 비해 적고 용량도 적다는 장점을 가지고 있다.
- 프로세스를 독립적인 환경에서 실행하므로써 프로세스의 관리와 확장에 용이해집니다.

# 도커란 ?

---

- **컨테이너** 기반 **가상화** 도구
- 도커는 컨테이너를 구축 , 배포 , 실행 하기위한 도구와 플랫폼을 제공합니다.
- 즉 , 사용자가 컨테이너 기술을 쉽게 사용할 수 있게하는 도구

# 도커의 이점

---

- **어플리케이션의 환경을 컨테이너로 캡슐화하여 어떤 환경에서든 일관되게 실행할 수 있다는 것 입니다.**
- 도커 컨테이너는 호스트 시스템과 독립적으로 실행되며 , 개발 환경과 운영 환경 사이의 차이로 인한 문제를 최소화할 수 있습니다.
- 도커 이미지는 어플리케이션 실행에 필요한 모든 구성 요소를 포함하고 있으며 , 이미지를 공유하고 재사용함으로써
    
    개발자들 간 협업과 어플리케이션의 이식성을 높힐 수 있습니다.
    
- 가상 머신에 비해 가볍고 빠릅니다.

# 도커 컴포즈

---

- 여러개의 컨테이너를 하나의 서비스로 정의하고 실행합니다.
- 컨테이너의 설정이 정의된 yml 파일을 읽고 도커 엔진을 통해 컨테이너를 실행합니다.
- 도커 컴포즈를 사용하는 이유는 여러 컨테이너의 생성을 한번에 편리하게 하기 위함.

# 나는 왜 도커를 사용했는가

---

- 환경에 종속 받지 않고 어플리케이션을 실행하기 위해.
- 어플리케이션 실행에 필요한 종속성에 의존하지 않고 실행만으로 어플리케이션을 실행하기위해