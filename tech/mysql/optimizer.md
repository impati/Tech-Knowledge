

## 개요

### 쿼리 실행 절차

MySQL 서버에서 쿼리가 실행되는 과정은 크게 세단계로 나눌 수 있다.

1. 사용자로부터 요청된 SQL 문장을 잘게 쪼개서 MySQL 서버가 이해할 수 있는 수준으로 분리한다.
2. SQL 파싱 정보를 확인하며 어떤 테이블을 읽고 어떤 인덱스를 이용해 테이블을 읽을지 선택한다.
3. 두 번째 단계에서 결정된 테이블의 읽기 순서나 선택된 인덱스를 이용해 스토리지 엔진으로부터 데이터를 가져온다.

첫 번쨰 단계를 SQL 파싱으로고하며 MySQL 서버의 SQL 파서가 모듈로 처리한다. 또한 이 단계에서 문법적으로 잘못된 부분이 걸러지고 SQL 파스 트리가 만들어진다.

두 번째 단계는 SQL 파스 트리를 참조하여 불필요한 조건 제거 및 복잡한 연산의 단순화 , 여러 테이블 조인이 있을 때 어떤 순서로 테이블을 읽을지 , 각 테이블에 사용된 조건과 인덱스 통계 정보를 이용해 사용할 인덱스를 결정 , 가져온 레코드들을 임시 테이블에 넣고 다시 한번 가공해야하는 지 결정을 처리한다. 이 단계는 최적화 및 실행 계획 수립 단계이며 MySQL 에서는 옵티마이저에서 처리한다. 이 단계를 통해 실행 계획이 만들어지며 

세 번째 단계에서 실행 계획으로 레코드들을 가져온다.

### 옵티마이저의 종류

옵티마이저는 데이터베이스 서버에서 두뇌와 같은 역할을 담당하며 **비용 기반 최적화 방법**과 **규칙 기반 최적화 방법**으로 나눌 수 있다. 

비용 기반 최적화는 쿼리를 처리하기 위한 다양한 방법을 만들고, 각 단위 작업의 비용 정보와 대상 테이블의 예측된 통계 정보를 이용해 실행 계획 별 비용을 산출하고 이 비용이 최소가 되는 처리 방식을 선택해 최종적으로 쿼리를 실행한다.

규칙 기반 최적화는 대상 테이블의 레코드 건수나 선택도를 고려하지 않고 옵티마이저에 내장된 우선순위에 따라 실행 계획을 수립하는 방식이다.이 방식에서는 통계 정보의 정보를 조사하지 않고 실행 계획을 수립하기 때문에 같은 쿼리에 대해서는 항상 같은 실행 계획을 만들어낸다.

## 기본 데이터 처리

MySQL 서버를 포함한 모든 RDMS 는 데이터를 정렬하거나 그루핑하는 등의 기본 데이터 가공 기능을 가지고 있다. 

### 풀 테이블 스캔과 풀 인덱스 스캔

풀 테이블 스캔은 인덱스를 사용하지 않고 테이블의 데이터를 처음부터 끝까지 읽어서 요청된 작업을 처리하는 작업을 의미한다. MySQL 옵티마이저는 테이블의 레코드 수가 작아서 인덱스를 통해 읽는 것보다 풀 테이블 스캔을 하는 편이 더 빠른 경우 , WHERE 절에 인덱스를 이용할 수 있는 적절한 조건이 없는 경우 , 조건 일치 레코드 건수가 너무 많은 경우에는 풀 테이블 스캔을 한다.

InnoDB 엔진은 특정 테이블의 연속된 데이터 페이지가 읽히면 백그라운드 스레드에 의해 **리드 어헤드** 작업이 시작된다. 리드 어헤드란 어떤 영역의 데이터가 앞으로 필요해지리라는 것을 예측해서 요청이 오기 전에 미리 디스크에서 읽어 InnoDB 버퍼 풀에 가져다 두는 것을 의미한다.

즉 , 풀 테이블 스캔이 실행되면 처음 몇 개의 데이터 페이지는 포그라운드 스레드가 페이지 읽기를 실행하지만 특정 시점부터는 읽기 작업을 백그라운드 스레드로 넘긴다. 포그라운드 스레드는 미리 버퍼 풀에 준비된 데이터를 가져다 사용하기만 하면 되므로 쿼리가 상당히 빨리 처리된다.

MySQL 서버에서는 innodb_read_ahead_threshold 시스템 변수를 이용해 InnoDB 엔진이 언제 리드 어헤드를 시작할지 임계점을 설정 할 수 있다.

리드 어헤드는 풀 테이블 스캔에서만 사용되는 것이 아니라 풀 인덱스 스캔에서도 동일하게 사용된다. 풀 테이블 스캔이 테이블을 처음부터 끝까지 스캔하는 것을 의미하듯이 풀 인덱스 스캔은 인덱스를 처음부터 끝까지 스캔하는 것을 의미한다.

## 병렬 처리

여기서 병렬 처리는 하나의 쿼리를 처리하기 위해 여러 스레드가 작업을 나누어 동시에 처리한다는 것을 의미한다.

아직 MySQL 서버에서는 쿼리를 여러 개의 스레드를 이용해 병렬로 처리하게 하는 힌트나 옵션은 없다. 8.0 버전 부터는 아무런 WHERE 조건 없이 단순히 테이블의 전체 건수를 가져오는 쿼리만 병렬로 처리할 수 있다.

## ORDER BY 처리

레코드 1 ~ 2건을 가져오는 쿼리를 제외하면 대부분의  SELECT 쿼리에서 정렬은 필수적으로 사용된다. 정렬을 처리하는 방법은 인덱스를 이용하는 방법과  쿼리가 실행될 때 Filesort 라는 별도의 처리를 이용하는 방법으로 나눌 수 있다.

MySQL 서버에서 인덱스를 이용하지 않고 별도의 정렬 처리를 수행했는지는 실행 계획의 Extra 칼럼에 “Using filesort” 메시지가 표시되는지 여부로 알 수 있따.

### 소트 버퍼

MySQL 은 정렬을 수행하기 위해 별도의 메모리 공간을 할당 받아서 사용하는데 이 메모리 공간을 소트 버퍼라고 한다. 소트 버퍼는 정렬이 필요한 경우에만 할당되며 버퍼의 크기는 정렬해야할 레코드 크기에 따라 가변적으로 증가하지만 최대 사용 가능한 소트 버퍼의 공간을 설정할 수 있다. 소트 버퍼를 위한 메모리 공간은 쿼리 실행이 완료되면 즉시 시스템으로 반납된다.

정렬해야할 레코드의 건수가 소트 버퍼로 할당된 공간보다 크다면 MySQL 은 정렬해야 할 레코드를 여러 조각으로 나눠서 처리하는데 이 과정에서 임시 저장을 위해 디스크를 사용한다. 메모리 소트 버퍼에서 정렬을 수행하고 그 결과를 임시로 디스크에 기록해 둔다. 그 다음 레코드를 가져와서 다시 정렬 후 반복적으로 디스크에 임시 저장한다. 이처럼 각 버퍼 크기만큼 정렬된 레코드를 다시 병합하면서 정렬을 수행해야한다. 이 병합 작업을 멀티 머지 라고 표현한다. 이 작업이 모두 디스크쓰기와 읽기를 유발하며 레코드 수가 많을수록 이 반복 작업의 횟수가 많아진다. 소트 버퍼를 크게 할당하는 것이 큰 도움이 되지는 않는다 더 큰 공간 할당에 대한 오버헤드가 발생하기 때문이다.

정렬을 위해 사용되는 소트 버퍼는 세션 메모리영역에 해당되어 여러 클라이언트가 공유할 수 있는 영역이 아니라 정렬 작업이 많을수록 소트 버퍼로 소비되는 메모리 공간이 커짐을 의미한다. 

### 정렬 알고리즘

레코드를 정렬할 때 레코드 전체를 소트 버퍼에 담을지 또는 정렬 기준 칼럼만 소트 버퍼에 담을지에 따라 싱글 패스 , 투 패스 2가지 정렬모드로 나눌 수 있다. 정렬을 수행하는 쿼리가 어떤 정렬 모드를 사용하는지는 옵티마이저 트레이스 기능으로 확인할 수 있다.

 

```sql
SET OPTIMIZER_TRACE="enabled=on",END_MARKERS_IN_JSON=on;

... 정렬 쿼리 후

SELECT * FROM INFORMATION_SCHEMA.OPTIMIZER_TRACE; 
```

출력된 내용에서 filesort_summary 색션의 sort_algorithm 필드에 정렬 알고리즘이 표시되고 , sort_mode 에는 다음 3가지가 표시된다.

- <sort_key,rowid> : 정렬 키와 레코드의 로우 아이디만 가져와서 정렬하는 방식
- <sort_key,addtional_fields> : 정렬 키와 레코드 전체를 가져와서 정렬하는 방식으로 레코드의 칼럼들은 고정 사이즈로 메모리 저장
- <sort_key,packed_addtional_fields> : 정렬 키와 레코드 전체를 가져와서 정렬하는 방식으로 레코드의 칼럼들은 가변 사이즈로 메모리 저장

이때 정렬 키와 레코드의 로우 아이디만 가져와 정렬하는 방식을 “투 패스” 정렬 방식이라고 하며 나머지는 “싱글 패스” 정렬 방식이라고한다.

싱글 패스 정렬 방식을 사용하면 정렬에 필요하지 않은 칼럼까지 모두 읽어 소트 버퍼에 담아 정렬을 수행한다. 그리고 정렬이 완료되면 정렬 버퍼의 내용을 그대로 클라이언트에게 전달한다.

반면 투 패스 정렬 방식은 정렬 대상 컬럼과 프라이머리 키 값만 소트 버퍼에 담아서 정렬을 수행하고 정렬된 순서대로 다시 프라이머리 키로 테이블을 읽어서 SELECT 칼럼을 가져오는 방식으로 싱글 패스 정렬 방식이 사용되기 이전부터 사용하던 방식이다. 8.0 에서도 특정 조건(레코드 크기가 큰 경우)에서는 투 패스 정렬 방식을 사용한다.

투 패스 정렬 방식은 테이블을 두 번 읽어야하기 때문에 불합리하지만 싱글 패스 정렬 방식은 더 많은 소트 버퍼 공간이 필요하거나 멀티 머지 작업이 더 많이 발생하여 디스크 읽기와 쓰기가 자주 발생한다. 그럼에도 불구하고 현재는 싱글 패스 정렬 방식을 주로 사용한다.

> SELECT 쿼리에서 꼭 필요한 컬럼만 조회하지 않고 모든 칼럼(*) 가져오도록 개발할 때가 많다. 하지만 이는 정렬 버퍼를 몇 배에서 몇십배까지 비효율적으로 사용할 가능성이 크다. SELECT 쿼리에서 꼭 필요한 컬럼만 조회하도록 쿼리를 작성하는 것이 좋다고 권장하는 이유는 바로 이런 이유 때문이다.

### 정렬 처리 방법

쿼리에 ORDER BY 가 사용되면 반드시 다음 3가지 처리 방법 중 하나로 정렬이 된다. 

- 인덱스를 사용한 정렬
- 조인에서 드라이빙 테이블만 정렬 ; Using filesort
- 조인에서 조인 결과를 임시 테이블로 저장한 후 정렬 ; Using temporary;Using filesort

먼저 옵티마이저는 정렬 처리를 위해 인덱스를 이용할 수 있을지 검토하고 인덱스를 이용할 수 있다면 별도의 Filesort 없이 인덱스를 순서대로 읽어서 결과를 반환한다. 하지만 인덱스를 사용할 수 없다면 WHERE 조건에 일치하는 레코드를 검색해 소트 버퍼에 저장하면서 정렬을 처리할 것이다. 이때 MySQL 옵티마이저는 정렬 대상 레코드를 최소화하기 위해 다음 2가지 방법을 선택한다.

- 조인에서 드라이빙 테이블만 정렬 ; Using filesort
    
    이 방법으로 정렬이 처리되려면 조인에서 첫 번째로 읽히는 테이블의 칼럼만으로 ORDER BY 절을 작성해야한다. 옵티마이저는 드라이빙 테이블만 검색해서 정렬을 먼저 수행하고 그 결과와 조인할 테이블을 조인하여 결과를 가져오는 방법으로 동작한다.
    
- 조인에서 조인 결과를 임시 테이블로 저장한 후 정렬 ; Using temporary;Using filesort
    
    쿼리가 여러 테이블을 조인하지 않고 하나의 테이블로부터 SELECT 해서 정렬하는 경우라면 임시 테이블이 필요하지 않다. 하지만 2개 이상의 테이블을 조인해서 그 결과를 정렬해야한다면 임시 테이블이 필요할 수도 있다. “조인에서 드라이빙 테이블만 정렬” 은 임시 테이블을 사용하지 않는다. 하지만 그 외 패턴의 쿼리에서는 항상 조인의 결과를 임시 테이블에 저장하고 그 결과를 다시 정렬하는 과정을 거친다.
    

일반적으로 조인이 수행되면서 레코드 건 수와 레코드의 크기는 거의 배수로 불어나기 때문에 가능하다면 드라이빙 테이블만 정렬한 다음 조인을 수행하는 방법이 효율적이다.

### 정렬 처리 방법의 성능 비교

주로 웹 서비스용 쿼리에서는 ORDER BY 와 함께 LIMIT 가 거의 필수로 사용되는 경향이 있다. 일반적으로 LIMIT 은 테이블이나 처리 결과의 일부만 가져오기 때문에 MySQL 이 처리해야할 작업량이 줄이는 역할을 하지만 ORDER BY , GROUP BY 같은 작업은 WHERE 조건을 만족하는 레코드를 LIMIT 건수만큼만 가져와서는 처리할 수 없다. 우선 조건을 만족하는 레코드를 모두 가져와 정렬을 수행하거나 그루핑 작업을 수행한 뒤에만 비로소 LIMIT 으로 건수를 제한할 수 있다. WHERE 조건이 아무리 인덱스를 잘 활용하도록 튜닝해도 ORDER BY , GROUP BY 때문에 쿼리가 느려지는 경우가 자주 발생한다.

쿼리에서 인덱스를 사용하지 못하는 정렬이나 , 그루핑 작업이 왜 느리게 작동할 수 밖에 없는지 스트리밍 처리와 버퍼링 처리를 통해 알아보자.

### 스트리밍 방식

스트리밍 방식은 서버 쪽에서 처리할 데이터가 얼마인지에 관계없이 조건에 일치하는 레코드가 검색될때마다 바로바로 클라이언트로 전송해주는 방식이다. 이 방식으로 쿼리를 처리할 경우 클라이언트는 쿼리를 요청하고 곧바로 원했던 첫 번째 레코드를 전달받는다.

쿼리가 스트리밍 방식으로 처리된다면 클라이언트는 MySQL 서버가 일치하는 레코드를 찾는 즉시 전달받기 때문에 동시에 데이터의 가공 작업을 시작할 수 있다. 또한 LIMIT 처럼 결과 건수를 제한하는 조건들은 쿼리의 전체 실행 시간을 상당히 줄여줄 수 있다.

### 버퍼링 방식

ORDER BY 나 GROUP BY 같은 처리는 쿼리의 결과가 스트리밍되는 것을 불가능하게 한다. 우선 WHERE 조건에 일치하는 모든 레코드를 가져온 후 , 정렬하거나 그루핑해서 차례대로 보내야하기 때문이다. MySQL 서버에서는 모든 레코드를 검색하거 정렬 작업을 하는 동안 클라이언트는 아무것도 하지 않고 기다려야하기 때문에 응답 속도가 느려진다. 

“정렬 처리 방법”에서 3가지 방법 가운데 인덱스를 사용한 정렬 방식만 스트리밍 형태의 처리이며 나머지는 모두 버퍼링된 후 정렬된다. 

> 스트리밍 처리는 어떤 클라이언트를 사용하느냐에 따라 다를 수 있는데 JDBC 라이브러리의 경우에는 내부 버퍼에 결과를 모두 담은 후 마지막 레코드가 전달될 때까지 기다렸다가 모든 결과를 전달받으면 그때서야 비로서 어플리케이션에 반환한다. 자체적으로 버퍼링을 하는 이유는 전체 처리 시간이 짧고 MySQL 서버와의 통신 횟수가 적어 자원 소모가 줄어들기 때문이다. JDBC 를 사용해도 전송방식을 스트리밍 방식으로 변경할 수 있다.
> 

> LIMIT 가 아무런 도움이 되지 못하는 것은 아니다. 정렬할 대상 레코드가 1000건인 쿼리에 LIMIT 10 이라는 조건이 있다면 정렬 하다가 멈추고 결과를 반환한다.
> 

## GROUP BY 처리

GROUPY BY 또한 쿼리가 스트리밍된 처리를 할 수 없게 하는 처리 중 하나이다. GROUPY BY 절이 있는 쿼리에서는 HAVING 절을 사용할 수 있는데 HAVING 절은 GROUPY BY 결과에 대해 필터링 역할을 수행한다. GROUPY BY 에 사용된 조건은 인덱스를 사용해서 처리될 수 없으므로 HAVING 절을 튜닝하려고 인덱스를 생성하거나 다른 방법을 고민할 필요없다.

GROUPY BY 작업도 인덱스를 사용하는 경우와 그렇지 못한 경우로 나눠 볼 수 있따. 인덱스를 이용할 때는 인덱스를 차례대로 읽는 인덱스 스캔 방법과 인덱스를 건너뛰면서 읽는 루스 인덱스 스캔이라는 방법으로 나뉜다.

그리고 인덱스를 사용하지 못하는 쿼리에서 GROUPY BY 작업은 임시 테이블을 사용한다.

### 인덱스 스캔을 사용하는 GROUPY BY

ORDER BY 경우와 마찬가지로 조인의 드라이빙 테이블에 속한 칼럼만 이용해 그루핑할 때 GROUPY BY 칼럼으로 이미 인덱스가 있다면 그 인덱스를 차례대로 읽으면서 그루핑 작업을 수행하고 그 결과로 조인을 처리한다.

GROUPY BY가 인덱스를 사용해서 처리된다 하더라도 그룹 함수 등의 그룹값을 처리해야 해서 임시 테이블이 필요할 때도 있다. GROUPY BY 가 인덱스를 통해 처리되는 쿼리는 이미 정렬된 인덱스를 읽는 것이므로 쿼리 실행 시점에 추가적인 정렬 작업이나 내부 임시 테이블은 필요하지 않는다. 이러한 그루핑 방식을 사용하는 쿼리의 실행 계획에서 Extra 컬럼에 별도로 GROUPY BY 관련 코멘트나 , 정렬 관련 코멘트가 표시되지 않는다.

### 루스 인덱스 스캔을 사용하는 GROUPY BY

루스 인덱스 스캔 방식은 인덱스의 레코드를 건너뛰면서 필요한 부분만 읽어서 가져오는 것을 의미하는데 , 옵티마이저가 루스 인덱스 스캔을 사용할 때는 실행 계획 Extra 칼럼에 “Using index for group-by” 코멘트가 표시된다. 

### 임시 테이블을 사용하는 GROUPY BY

GROUPY BY의 기준 칼럼이 드라이빙 테이블에 있든 드리븐 테이블에 있든 관계없이 인덱스를 전혀 사용하지 못할 때는 이 방식으로 처리된다. 8.0 부터는 GROUPY BY 가 필요한 경우 내부적으로 GROUPY BY절의 칼럼들로 구성된 유니크 인덱스를가진 임시 테이블을 만들어서 중복제거와 함수 연산을 수행한다. 하지만 GROUPY BY 와 ORDER BY 가 같이 사용되면 명시적으로 정렬 작업을 실행한다. 이때 Extra 칼럼에는 Using temporary 와 함께 Using filesort 가 표시된다.


## DISTINCT 처리

특정 칼럼의 유니크한 값만 조회하려면 SELECT 쿼리에 DISTINCT 를 사용한다.  DISTINCT 는 MIN(),MAX() 또는 COUNT() m같은 집합 함수와 함께 사용되는 경우와 집합 함수가 없는 경우의 2가지로 구분할 수 있다.

### SELECT DISTINCT …

단순히 SELECT 되는 레코드 중에서 유니크한 레코드만 가져오고자 하면 SELECT DISTINCT 형태의 쿼리 문장을 사용한다. 이 경우에는 GROUP BY 와 동일한 방식으로 처리한다. 

### 집합 함수와 함께 사용된 DISTINCT

COUNT()  또는 MAX(), MIN() 같은 집합 함수 내에서 DISTINCT 가 사용될 수 있는데 이 경우에는 일반적으로 SELECT DISTINCT 와 다른 형태로 해석된다. SELECT 쿼리에서 DISTINCT 는 조회하는 모든 칼럼의 조합이 유니크한 것들만 가져온다. 하지만 집합 함수 내에서 사용된 DISTINCT 는 그 집합함수의 인자로 전달된 칼럼값이 유니크한 것들을 가져온다.

그리고 이 쿼리를 해결하기 위해서 임시 테이블을 사용한다.하지만 실행 계획에는 임시 테이블을 사용한다는 메시지는 표현되지 않는다. 

## 내부 임시 테이블 활용

MySQL 엔진이 스토리지 엔진으로부터 받아온 레코드를 정렬하거나 그루핑할 때는 내부적으로 임시 테이블을 사용한다.  임시 테이블은 처음에는 메모리에 생성되었다가 테이블의 크기가 커지면 디스크로 옮긴다. 그리고 이 임시 테이블은 다른 세션이나 다른 쿼리에서 볼 수 없다. 그리고 이 임시 테이블은 쿼리 처리가 완료되면 자동으로 삭제된다.

### 임시 테이블이 필요한 쿼리

다음과 같은 패턴의 쿼리는 MySQL 엔진에서 별도의 데이터 가공 작업을 필요로 하므로 대표적으로 내부 임시 테이블을 생성하는 케이스이다. 이 밖에도 인덱스를 사용하지 못할 때는 내부 임시 테이블을 생성해야할 때가 많다.

- ORDER BY 와 GROUP BY 에 명시된 컬림이 다른 쿼리
- ORDER BY 나 GROUP BY 에 명시된  칼럼이 조인의 순서상 드라이빙 테이블이 아닌 쿼리
- DISTINCT 와 ORDER BY 가 동시에 쿼리에 존재하는 경우 또는 DISTINCT 가 인덱스로 처리되지 못할 때
- UNION 이나 DISTINCT UNION 가 사용된 쿼리
- 쿼리의 실행 계획에서 SELECT_TYPE 이 DERIVED 쿼리

대게는 임시 테이블을 사용하는 경우 Using temporary 라는 메시지가 표시된다(그렇지 않은 케이스도 있다). 일반적으로 유니크 인덱스가 있는 내부 임시 테이블은 그렇지 않은 쿼리보다 처리 성능이 상당히 느리다.

## 고급 최적화

MySQL 서버의 옵티마이저가 실행 계획을 수립할 때 통계 정보와 옵티마이저 옵션을 결합해서 최적의 실행 계획을 수립한다. 옵티마이저 옵션은 크게 `조인 관련된 옵티마이저 옵션`과 `옵티마이저 스위치`로 구분할 수 있다.

## 옵티마이저 스위치 옵션

MySQL 서버의 고급 최적화 기능들을 활성화할지를 제어하는 용도로 사용된다.

`default`,`on`,`off` 중 하나로 설정 가능하고 글로벌 혹은 세션별 설정할 수 있다.

### MRR과 배치 키 엑세스

MRR 은 Multi-Range Read 를 줄여서 부르는 이름인데 , 메뉴얼에서는 DS-MRR(Disk Sweep Multi-Range Read) 이라고도 한다. MySQL 서버에서 지금까지 지원하던 조인 방식은 드라이빙 테이블의 레코드를 한 건 읽어서 드리븐 테이블의 일치하는 레코드를 찾아서 조인을 수행하는 것이었다. 이를 **네스티드 루프조인** 이라고 한다.

MySQL 서버의 내부 구조상 조인 처리는 MySQL 엔진이 처리하지만 , 실제 레코드를 검색하고 읽는 부분은 스토리지 엔진이 담당한다. 이때 드라이빙 테이블의 레코드 건별로 드리븐 테이블의 레코드를 찾으면 레코드를 찾고 읽는 스토리지 엔진에서는 아무런 최적화를 수행할 수가 없다.

이 같은 단점을 보완하기 위해 MySQL 서버는 조인 대상 테이블 중 하나로부터 레코드를 읽어서 조인 버퍼에 버퍼링을 한다. 즉 드라이빙 테이블의 레코드를 읽어서 드리븐 테이블과의 조인을 즉시 실행하지 않고 조인 대상을 버퍼링하는 것이다. 조인 버퍼에 레코드가 가득 차면 비로소 MySQL 엔진은 버퍼링된 레코드를 스토리지 엔진으로 한 번에 요청한다. 이렇게 함으로써 스토리지 엔진은 읽어야할 레코드들을 데이터 페이지에 정렬된 순서로 접근해서 디스크 페이지 읽기를 최소화할 수 있는 것이다. 이러한 읽기 방식을 MRR 이라고하며 MRR 을 응용해서 실행되는 조인 방식을 BKA(Batched Key Access) 조인이라고 한다. BKA 조인 최적화는 기본적으로 비활성화되어 있는데 BKA 의 단점도 존재하기 때문이다. 쿼리 특성 상 도움이 되는 경우가 있지만 부가적인 정렬 작업이 필요해지면서 오히려 성능에 안 좋은 영향을 미치는 경우가 있다.

### 블록 네스티드 루프 조인

MySQL 서버에서 사용되는 대부분의 조인은 네스티드 루프 조인인데 , 조인의 연결 조건이 되는 칼럼에 모두 인덱스가 있는 경우 사용되는 조인 방식이다.

네스티드 루프 조인과 블록 네스티드 루프 조인의 가장 큰 차이는 조인 버퍼가 사용되는지 여부와 조인에서 드라이빙 테이블과 드리븐 테이블이 어떤 순서로 조인되느냐다. 조인 알고리즘에서 BLOCK 이라는 단어가 사용되면 조인용으로 별도의 버퍼가 사용됐다는 것을 의미하는데 , 조인 쿼리 실행 계획에서 Using Join Buffer 라는 문구가 표시된다.

**조인은 드라이빙 테이블에서 일치하는 레코드의 건 수만큼 드리븐 테이블을 검색하면서 처리된다. 즉 드라이빙 테이블은 한 번에 쭉 읽지만 드리븐 테이블은 여러 번 읽는다는 것을 의미한다. 그래서 드리븐 테이블을 검색할 때 인덱스를 사용할 수 없는 쿼리는 상당히 느려지며 , 옵티마이저는 최대한 드리븐 테이블의 검색이 인덱스를 사용할 수 있게 실행 계획을 수립한다.**

그런데 어떤 방식으로도 드리븐 테이블의 풀 테이블 스캔이나 인덱스 풀 스캔을 피할 수 없다면 옵티마이저는 드라이빙 테이블에서 읽은 레코드를 메모리에 캐시한 뒤 드리븐 테이블과 이 메모리 캐시를 조인하는 형태로 처리한다.이때 사용되는 메모리의 캐시를 조인 버퍼라고한다. 

조인 버퍼가 사용되는 쿼리는 조인의 순서가 거꾸로인 것 처럼 실행된다. 드라이빙 테이블의 결과를 조인 버퍼에 담아두고 드리븐 테이블을 먼저 읽고 조인 버퍼에서 일치하는 레코드를 찾는 방식으로 처리하기 때문이다.

> 8.0 이후 특정 버전부터는 해시 조인 알고리즘이 도입됐으며 ,  블록 네스티드 루프 조인을 사용하지 않고 해시 조인 알고리즘이 대체되어 사용된다고 한다.

### 인덱스 컨디션 푸시다운(index_condition_pushdown)

```sql
ALTER TABLE employees ADD INDEX ix_lastname_firstname (last_name , first_name);
```

위와 같은 인덱스가 존재하고 다음 쿼리를 실행할 때 스토리지 엔진이 몇 건의 레코드를 읽는지 살펴보자. 

```sql
SELECT * FROM employees WHERE last_name = 'Action' AND first_name LIKE '%sal';
```

last_name 조건은 인덱스로 레인지 스캔으로 일을 수 있다. 하지만 first_name LIKE ‘%sal’구문은 인덱스 레인지 스캔으로 검색해야할 범위를 좁힐 수가 없는 구문이다. 결국 이는 데이터를 모든 읽은 후 사용자가 원하는 결과인지 하나씩 비교해보는 필터링을 통해 만족된다. 이러한 방법은 “Using where” 으로 표시되며 “Using where”  이 의미하는 바는 스토리지 엔진이 읽어서 반환해준 레코드가 인덱스를 사용할 수 없는 WHERE 조건에 일치하는지 검사하는 과정을 의미한다. 다시말해 스토리지 엔진으로부터 100만건을 가져왔지만 결국 1건만 만족했다면 999,999건의 레코드 읽기가 불필요했던 것이다. 이러한 일이 발생하는 이유는 인덱스 비교는 스토리지 엔진이 하지만 테이블의 레코드에서 조건을 비교하는 작업은 MySQL 엔진이 하기 때문이다.  

5.6 버전부터는 인덱스 컨디션 푸시다운이라는 기능이 도입되어 인덱스를 범위 제한 조건으로 사용하지 못한다고 하더라도 인덱스에 포함된 칼럼의 조건이 있다면 모두 같이 모아서 스토리지 엔진으로 전달할 수 있게 핸들러 API 가 개선되었다. 이를 통해 최대한 필터링까지 완료해서 꼭 필요한 레코드 1건만 테이블 읽기를 수행할 수 있게 되었다.인덱스 컨디션 푸시다운을 사용했다면 Using where 가 없어지고 Using index condition 이 출력되는 것을 볼 수 있다. 이는 쿼리 성능을 몇 배에서 몇십 배로 향상될 수도 있는 중요한 기능이다.

### 인덱스 확장(use_index_extensions)

use_index_extensions 옵티마이저 옵션은 InnoDB 스토리지 엔진을 사용하는 테이블에서 세컨더리 인덱스에 자동으로 추가된 프라이머리 키를 활용할 수 있게 할지를 결정하는 옵션이다. 과거에는 프라이머리 키를 제대로 활용하지 못했지만 현재는 프라이머리 키에 대한 인덱스가 존재한다는 것을 인지하고 실행 계획을 수립한다.

이는 `secondary_index_key , primary_index_key` 조합의 인덱스를 이용하는 것과 흡사하게 동작한다.

### 인덱스 머지(index_merge)

인덱스를 이용해 쿼리를 실행하는 경우,  대부분 옵티마이저는 테이블별로 하나의 인덱스만 사용하도록 실행 계획을 수립한다. 하지만 인덱스 머지 실행 계획을 사용하면 하나의 테이블에 대해 2개 이상의 인덱스를 이용해 쿼리를 처리한다. 

일반적으로 쿼리에서 한 테이블에 대한 WHERE 조건이 여러 개 있더라도 하나의 인덱스에 포함된 칼럼에 대한 조건만으로 인덱스를 검색하고 나머지 조건은 읽어온 레코드에 대해서 체크하는 형태로만 사용된다. 이처럼 하나의 인덱스만 사용해서 작업 범위를 충분히 줄일 수 있는 경우라면 테이블별로 하나의 인덱스만 활용하는 것이 효율적이다. 하지만 쿼리에 사용된 각각의 조건이 서로 다른 인덱스를 사용할 수 있고 그 조건을 만족하는 레코드 건수가 많을 것으로 예상될 때 MySQL 서버는 인덱스 머지 실행 계획을 수립한다. 인덱스 머지 실행 계획은 다음과 같이 3개의 세부 실행 계획으로 나누어 볼 수 있다.

### 인덱스 머지 - 교집합

```sql
SELECT * FROM employees
WHERE first_name = 'impati' AND emp_no BETWEEN 10000 AND 20000;
```

위 쿼리에서 first_name , emp_no 에 대한 각각의 인덱스를 가지고 있다고 했을 때 모두 인덱스를 사용해 결과를 교집합해서 반환할 수 도 있다. 그럴 경우 Extra 칼럼에 “Using intersect” 라고 표시된다.

각 인덱스 중 하나만으로도 충분히 효율적으로 처리할 수 있었다면 2개의 인덱스 모두를 사용하지 않았을 것이다. 옵티마이저는 각각의 조건에 일치하는 레코드 건수를 예측해 본 결과 모두 상대적으로 많은 레코드를 가져와야한다는 것을 알게 된 것이다. 

### 인덱스 머지 - 합집합

인덱스 머지의 Using union 은 WHERE 절에 사용된 2개 이상의 조건이 각각의 인덱스를 사용하되 OR 연산자로 연결된 경우에 사용되는 최적화이다. 두개 이상의 인덱스로 검색한 결과를 Union 알고리즘으로 병합했다는 것을 의미한다.

### 세미 조인

다른 테이블과 실제 조인을 수행하지는 않고, 단지 다른 테이블에 조건에 일치하는 레코드가 있는지 체크하는 형태의 쿼리를 세미 조인이라고한다. 

```sql
SELECT * FROM employees e 
WHERE e.emp_no IN(SELECT de.emp_no FROM dept_emp de.from_date = '1995-01-01');
```

MySQL 서버에서 세미 조인 최적화 기능이 없었을 때에 위의 세미 조인 쿼리 실행 계획은 employees 테이블을 풀 스캔하면서 한 건 한 건 서브쿼리의 조건에 일치하는지 비교했다. 그래서 57건만 읽어도 될 쿼리를 30만 건 넘게 읽어서 처리되었었다.

8.0부터 세미 조인 쿼리의 성능을 개선하기 위해 다음과 같은 최적화 전략이 있으며 MySQL 서버 메뉴얼에서는 아래 최적화 전략을 모아 세미 조인 최적화라고 부른다.

- 테이블 풀 아웃(Table pull-out)
- 중복 제거(Duplicated Weed-out)
- First Match
- Loose Scan
- Materialization

쿼리에 사용되는 테이블과 조인 조건의 특성에 따라 옵티마이저는 사용 가능한 전략들을 선별적으로 사용한다. Table pull-out 최적화 전략은 항상 세미 조인보다는 좋은 성능을 내기 때문에 별도로 제어하는 옵션을 제공하지 않는다.

### 테이블 풀-아웃(Table Pull-out)

Table Pull-out 최적화는 세미 조인의 서브쿼리에 사용된 테이블을 아우터 쿼리로 끄집어낸 후에 쿼리를 조인 쿼리로 재작성한 형태의 최적화이다. 이는 서브쿼리 최적화가 도입되기 전에 수동으로 쿼리를 튜닝하던 방법 중 하나였다고한다.

### 퍼스트 매치(First Match)

퍼스트 매치 전략은 IN(subquery) 형태의 세미 조인을 EXISTS(subquery) 형태로 튜닝한 것과 비슷한 방법으로 실행된다.

### 루스 스캔(loosescan)

세미 조인 서브쿼리 최적화의 루스 스캔은 인덱스를 사용하는 GROUP BY 최적화 방법에서 살펴본 Using index for group by 의 루스 인덱스 스캔과 비슷한 읽기 방식을 사용한다. 

### 구체화(Materialization)

세미 조인에 사용된 서브쿼리를 통쨰로 구체화해서 쿼리를 최적화한다는 의미이다. 쉽게 표현하면 임시 테이블을 생성한다는 것을 으미한다.

### 중복제거(Duplicated Weed-out)

중복 제거는 세미 조인 서브쿼리를 일반적인 INNER JOIN 쿼리로 바꿔서 실행하고 마지막에 중복된 레코드를 제거하는 방법으로 처리되는 최적화 알고리즘이다. INNER JOIN + GROUP BY 처럼 실행하는 것과 동일한 작업으로 쿼리를 처리한다.Extra 칼럼에 `Start temporary` 와 `End temporary` 가 표기된다.

### 컨디션 팬아웃

조인을 실행할 때 테이블의 순서는 쿼리의 성능에 매우 큰 영향을 미친다. 예를 들어 A 테이블과 B 테이블을 조인할 때 A 테이블에는 조건에 일치하는 레코드가 1만건이고 B 테이블에는 일치하는 레코드가 10건이라고 해보자. 이때 A 테이블을 조인 드라이빙 테이블로 결정하면 B 테이블을 1만번 읽어야한다. 이때 B 테이블의 인덱스를 이용해 조인한다고 하더라도 레코드를 읽을 때마다 B 테이블의 인덱스를 구성하는 B-Tree 의 루트노드부터 검색을 실행해야한다. 그래서 MySQL 옵티마이저는 여러 테이블이 조인되는 경우 가능하다면 일치하는 레코드 건수가 적은 순서대로 조인을 실행한다. 컨디션 팬아웃 최적화 기능을 활성화하면 MySQL 옵티마이저는 더 정교한 계산을 거쳐 실행 계획을 수립한다. 그에 따라 쿼리의 실행 계획 수립에 더 많은 시간과 컴퓨팅 자원을 사용하게 되는데 쿼리가 크게 잘못된 선택을 하지 않거나 간단하다면 큰 도움이 되지는 않을 수 있다.

### 파생 테이블 머지

예전 버전의 MySQL 서버에서는 다음과 같이 FROM 절에 사용된 서브쿼리는 먼저 실행해서 그 결과를 임시터에블로 만든다음 외부 쿼리 부분을 처리했다.

```kotlin
 SELECT * FROM (SELECT * FROM employees WHERE first_name = 'Matt') derived_table
 WHERE derived_table.hire_date = '1986-04-03'
```

이렇게 FROM 절에 사용된 서브쿼리를 파생 테이블이라고 부른다.

파생 테이블의 경우에는 임시 테이블을 생성하고 레코드를 읽고 임시 테이블로 INSERT 한다. 레코드를 복사하고 읽는 과정의 오버헤드가 추가되며 이 레코드 건수가 많아짐에 따라 디스크로 다시 기록되어야한다. 그래서 임시 테이블이 메모리에 상주할 만큼 크기가 작다면 성능에 크게 영향을 주지 않았겠지만 많아진다면 더 느려질 것이다.

5.7 버전부터는 파생 테이블로 만들어지는 서브쿼리를 외부 쿼리와 병합해서 서브 쿼리부분을 제거하는 최적화가 되입됐는데 이를 파생 테이블 머지라고한다.

### 스킵 스캔

인덱스의 핵심은 값이 정렬돼 있다는 것이며 이로 인해 인덱스를 구성하는 칼럼의 순서가 매우 중요하다.

예를 들어 (A,B,C) 칼럼으로 구성된 인덱스가 있을 때 쿼리의 WHERE 절에 A 와 B 컬럼에 대한 조건이 있다면 이 쿼리는 A,B 칼럼까지만 인덱스를 활용할 수 있도, A 와 관련된 조건이 있다면 A 칼럼까지만 인덱스를 구성할 수 있다. 그런데 WHERE 절에 B,C 칼럼에 대한 조건을 가지고 있다면 이 쿼리는 인덱스를 이용할 수 없다. 인덱스 스캡 스캔은 제한적이지만 인덱스의 이런 제약 사항을 뛰어넘을 수 있는 최적화 기법이다.

이 기법은 8.0 부터 인덱스 스킵 스캔 최적화가 도입되어 사용가능하다. 이 기능은 인덱스의 선행 칼럼이 조건절에 사용되지 않더라도 후행 칼럼의 조건만드로 인덱스를 이용한 쿼리 성능 개선이 가능하다. 하지만 선행 칼럼의 카디널리티가 높다면 비효율적일 수 있다. 그래서 옵티마이저는 인덱스의 선행 칼럼이 소수의 유니크한 값을 가질 수 있을때만 인덱스 스캡 스캔 최적화를 사용한다.

### 해시 조인

MySQL 8.0.18 버전부터는 해시 조인이 추가로 지원되기 시작했다. MySQL 해시 조인은 첫번째 레코드를 찾는는 시간이 오래 걸리지만 최종 레코드를 찾는데 까지는 시간이 많이 걸리지 않는다. 반면 네스티드 루프 조인은 마지막 레코드를 찾는데까지 오래걸리지만 첫 번째 레코드를 찾는 것은 상대적으로 빠르다.

이러한 이유로 MySQL  서버는 주로 조인 조건의 칼럼이 인덱스가 없다거나 조인 대상 테이블 중 일부 레코드 건 수가 매우 적은 경우 등에서만 해시 조인 알고리즘을 사용하도록 설계돼 있다. 해시 조인 방식은 Extra 칼럼에 “hash join” 라고 표시된다.

일반적으로 해시 조인은 빌드 단계와 프로브 단계로 나뉘어 처리된다.

빌드 단계에서는 조인 대상 테이블 중에서 레코드 건 수가 적어서 해시 테이블로 만들기 용이한 테이블을 골라서 메모리에 해시 테이블을 생성하는 작업을 수행한다.  빌드 단계에서 해시 테이블을 만들 때 사용되는 원본 테이블을 빌드 테이블이라고한다. 

프로브 단계에서는 나머지 테이블의 레코드를 읽어서 해시 테이블의 일치 레코드를 찾는 과정을 의미한다. 이때 읽는 나머지 테이블을 프로브 테이블이라고한다.

### 인덱스 정렬 선호

옵티마이저는 ORDER BY 또는 GROUP BY 를 인덱스를 사용해서 처리 가능한 경우 쿼리의 실행 계획에서 이 인덱스의 가중치를 높이 설정해서 실행된다.

```kotlin
 SELECT * FROM employees WHERE hire_date BETWEN ... 
 ORDER BY emp_no;
```

이때 hire_date 로 설정된 인덱스를 이용해 일치하는 레코드를 찾아 정렬해서 결과를 반환하는 방법과 order by 에서 사용되는 프라이머리 emp_no 키를 정순으로 읽으면서 hire_date 칼럼의 조건에 일치하는지 비교 후 결과를 반환하는 방법이 있는데 전자의 방법이 후자의 방법보다 더 빠른 경우에도 후자의 방법을 선택하는 경우가 있을 수도 있다. 이는 인덱스 정렬 선호 때문이다.

8.0.21 부터는 ORDER BY 또는 GROUP BY 를 위한 인덱스에 너무 많은 가중치를 부여하지 않도록 prefer_ordering_index 옵티마이저 옵션이 추가됐다.

## 쿼리 힌트

옵티마이저에게 쿼리의 실행 계획을 어떻게 수립해야 할지 알려주는 목적으로 다양한 힌트가 제공된다.

MySQL 서버에서 사용 가능한 쿼리 힌트는 2가지로 구분할 수 있다. `인덱스 힌트` , `옵티마이저 힌트` 

## STRAIGHT_JOIN

STRAIGHT_JOIN 은 `SELECT` , `UPDATE` ,`DELETE`   쿼리에서 여러 개의 테이블이 조인되는 경우 조인 순서를 고정하는 역할은 한다. 

일반적으로 조인을 하기 위한 칼럼들의 인덱스 여부로 조인의 순서가 결정되며, 조인 칼럼의 인덱스에 아무런 문제가 없는 경우에는 WHERE 조건을 만족하는 레코드가 적은 테이블을 드라이빙 테이블로 선택한다. 그래서 옵티마이저가 실행계획을 잘못 수립하는 경우라면 STRAIGHT_JOIN 힌트를 이용하는 것이 좋다.

## 인덱스 힌트

`STRAIGHT_JOIN` , `USE INDE` 등을 포함한 인덱스 힌트들은 모두 MySQL 서버에 옵티아미어 힌트가 도입되기 전에 사용되던 기능들이다. 이들은 모두 SQL 문법에 맞게 사용해야하기 때문에 사용하게 된다면 ANSI-SQL 표준 문법을 준수하지 못하는 단점이 있다. 또한 인덱스 힌트는 SELECT 문과 UPDATE 명령에서만 사용할 수 있다.

대체로 옵티마이저는 어떤 인덱스를 사용해야할지 무난하게 잘 선택하는 편이다. 하지만 3~4개 이상의 칼럼을 포함하는 비슷한 인덱스가 여러 개 존재하는 경우에는 가끔 실수하는데 이런 경우 강제로 특정 인덱스를 사용하도록 힌트를 추가한다.

인덱스 힌트는 3가지 종류가 있다. 

- USE INDEX : 옵티마이저에게 특정 테이블의 인덱스를 사용하도록 권장하는 힌트 정도이다.즉 , 항상 힌트로 준 인덱스를 사용하는 것은 아니다.
- FORCE USE INDEX : USE INDEX 보다 옵티마이저에게 미치는 영향이 더 강한 힌트 정도.
- IGNORE INDEX : 특정 인덱스를 사용하지 못하도록 하는 힌트이다.때로는 테이블 풀 스캔을 유도하기 위해 사용한다.

### 옵티마이저 힌트

8.0 버전에서 사용가능한 힌트는 종류가 매우 다양하며 힌트가 미치는 영향 범위도 다양하다.

### MAX_EXECUTION_TIME

옵티마이저 힌트 중에서 유일하게 쿼리의 실행 계획에 영향을 미치는 힌트이며, 단순히 쿼리의 최대 실행 시간을 설정하는 힌트이다. MAX_EXECUTION_TIME 힌트에는 밀리초 단위의 시간을 설정하는데 쿼리가 지정된 시간을 초과하면 쿼리는 실패하게 된다.

```kotlin
SELECT /*+ MAX_EXECUTION_TIME(100) */ *
FROM employees
ORDER BY last_name LIMIT 1
```

### SET_VAR

SER_VAR 힌트는 실행 계획을 바꾸는 용도뿐만 아니라 조인 버퍼나, 정렬용 버퍼의 크기를 일시적으로 증가시켜 대용량 처리 쿼리의 성능을 향상시키는 용도로 사용할 수 있다. 다양한 형태의 시스템 변수 조정을 할 수 있으면서 , 그렇다고 모든 시스템 변수는 조정할 수는 없다는 것도 기억해두자.

### 이외

이외에도 여러가지 최적화 방법을 사용할지말지를 결정할 수 있는 힌트들이 있는데 필요할 때 찾아보자.


## Reference
- [Real MySQL 8.0(1권)](https://www.yes24.com/Product/Goods/103415627)
