# Join 이란?
 

조인이란 한 테이블의 행을 다른 테이블의 행에 연결하여 두 개 이상의 테이블을 결합하는 연산이다.

- INNER JOIN
    
    두 테이블 간의 공통 속성을 기반으로 매칭되는 요소만 결합하는 연산이다.
    
- [LEFT | RIGHT | FULL ] OUTER JOIN
    
    두 테이블 간 매칭되는 요소 뿐만 라니라 매칭 되지 않는 레코드도 결합하는 연산으로 결합할 테이블의 레코드가 없는 경우에 NULL 값으로 반환합니다. 
    
- CROSS JOIN
    
    CROSS JOIN 키워드로  (조합)모든 경우의 수를 전부 결합하여 결과로 반환합니다.
    
- SELF JOIN
    
    하나의 테이블 내에서 자기 자신을 조인하는 경우로 같은 테이블 내에서 서로 다른 레코드들을 연결하여 결과를 반환합니다.
