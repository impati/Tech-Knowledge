
# 무결성 제약 조건
무결성 제약조건이란 데이터의 일관성과 신뢰성을 유지하게 위해 데이터 조작시 동반되는 제약조건을 말합니다.


### 개체 제약 조건 (기본키 제약)
릴레이션의 기본키를 구성하는 속성은 NULL 값이나 중복된 값을 가질 수 없습니다.
    
  
### 참조 제약 조건 (외래키 제약)
외래키 값은 NULL 이거나 참조하는 릴레이션의 기본키 값과 동일해야합니다.
    
  
### 도메인 제약 조건
속성값들은 정의된 도메인에 속한 값이어야합니다.
    
### 고유 제약 조건
특정 속성에 대해 고유한 값을 가지도록 조건이 주어진 경우 , 릴레이션의 각 튜플이 가지는 속성 값들은 서로 달라아합니다.
    
### NULL 제약 조건
릴레이션의 특정 속성 값이 NULL 을 가지지 않도록 조건이 주어진 경우 릴레이션의 각 튜플의 속성값은 NULL 을 가질 수 없습니다.
  
### 키 제약 조건
각 릴레이션은 최소한 한 개 이상의 키가 존재해야합니다.
