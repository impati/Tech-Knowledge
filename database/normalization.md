# 이상현상

**잘못 설계된 테이블로 데이터 삽입 ,삭제 , 수정시 불필요한 값을 삽입해야 하거나 , 연쇄 삭제가 일어나거나 , 데이터 일관성이 깨지는 등의 현상이 발생하는 경우를 이상현상이라고 합니다.**

# 정규화

**관계형 데이터베이스에서 중복을 최소화하고 이상현상을 없애기 위해 데이터를 구조화하는 작업입니다.**

- 함수 종속성 : A 속성을 알면 다른 속성 B 값이 유일하게 정해지는 의존 관계에서 B 는 A에 종속한다. 라고 표현합니다. 다시 말해 B가 A 에 의해 결정될 때 함수적 종속성을 가진다고 말합니다.
- 완전 함수 종속성 : **A 속성 집합의 모든 속성**들로 인해 **B가 결정**될 때 완전 함수 종속성이라고 합니다. 만약 A 의 부분 속성 집합이 B 를 결정하는 경우 부분 함수 종속성을 가진다고 합니다.
- 이행적 종속성 :  A → B → C ,  A → C 가 성립되는 함수 종속성
1. **제 1 정규화**
    
    모든 릴레이션의 속성은 원자값을 가진다.
    
2. **제 2 정규화** 
    
    제 1 정규형을 만족하면서 기본키가 아닌 속성이 기본키에 완전 함수 종속성을 만족할 때 제 2 정규형이라고 합니다.
    
3. **제 3 정규화**
    
    제 2 정규형을 만족하면서 기본키가 아닌 속성이 기본키에 비이행적 종속할 때 제 3 정규형이라고합니다.
    
4. **BCNF 정규화**
    
    제 3 정규형을 만족하면서 모든 결정자 X 가 후보키라면 BCNF 정규형이라고합니다. 즉,  후보키만이 속성을 결정할 때 BCNF 정규형이라고합니다.
    

## 장점

- 데이터 변경시 이상현상이 발생하는 문제점을 해결할 수 있습니다.
- 확장성 있는 설계가 가능해지며 변경에 유연하게 대응할 수 있습니다.

## 단점

- 릴레이션 분해로 인한 JOIN 연산이 많아집니다 .이 때문에 성능 저하로 이어질 가능성이 있습니다.

## 반 정규화 (De - normalization)

반 정규화는 정규화된 앤티티 , 속성 , 관계를 시스템 성능 향상 및 개발과 운영 단순화를 위해 중복 통합 ,분리 등을 수행하는 데이터 모델링 기법 중 하나로 , 조인 연산으로 인한 성능 저하나 칼럼을 계산하여 조회할 때 성능이 저하될 것이 예상되는 경우 반정규화를 수행합니다.