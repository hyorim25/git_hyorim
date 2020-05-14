# sql 활용
## 표준 조인

#### 1. STANDARD SQL 개요
현재 기업형 DBMS는 순수 관계형 데이터베이스가 아닌 객체 지원 기능이 포함된 객체관계형(Object Relational) 데이터베이스를 대부분 사용하고 있다. 
현재 우리가 사용하는 많은 시스템의 두뇌 역할을 하는 관계형 데이터베이스를 유일하게 접속할 수 있는 언어가 바로 SQL이다. 사용자와 개발자 입장에서는 SQL의 진화 및 변화가 가장 큰 관심 내용인데, 초창기 SQL의 기본 기능을 정리했던 최초의 SQL-86 표준과 관계형 DBMS의 폭발적인 전성기를 주도했던 ANSI/ISO SQL2 세대를 지나면서 많은 기술적인 발전이 있었다. 그러나, ANSI/ISO SQL2의 경우 표준 SQL에 대한 명세가 부족한 부분이 있었고, DBMS 벤더 별로 문법이나 사용되는 용어의 차이가 너무 커져서 상호 호환성이나 SQL 학습 효율이 많이 부족한 문제가 발생하였다. 이에 향후 SQL에서 필요한 기능을 정리하고 호환 가능한 여러 기준을 제정한 것이 1999년에 정해진 ANSI/ISO SQL3이다. 이후 가장 먼저 ANSI/ISO SQL3의 기능을 시현한 것이 Oracle의 8i/9i 버전이라고 할 수 있다. 참고로 2003년에 ANSI/ISO SQL 기준이 소폭 추가 개정되었고 현재 사용되는 데이터베이스는 대부분 SQL-2003 표준을 기준으로 하고 있다. 다른 벤더의 DBMS도 2006년 이후 발표된 버전에서 ANSI/ISO SQL-99와 SQL-2003의 핵심적인 기능은 만족스러운 수준으로 구현된 것으로 평가 받고 있다. 마지막으로 2008년에 진행된 추가 개정 내용은 아직 사용자 레벨에 큰 영향을 미치지 않고 있다. 아직도 벤더별로 일부 기능의 개발이 진행 중인 경우도 있고 벤더별 특이한 기술 용어는 여전히 호환이 안 되고 있지만, ANSI/ISO SQL 표준을 통해 STANDARD JOIN을 포함한 많은 기능이 상호 벤치마킹하고 발전하면서 DBMS 간에 평준화를 이루어 가고 있다고 볼 수 있다. 예를 들면, IBM DB2나 SYBASE ASE DBMS는 과거 버전부터 CASE 기능이나 FULL OUTER JOIN 기능을 지원하였지만, Oracle DBMS는 양쪽(FULL) OUTER JOIN의 경우 (+) 표시를 이용한 두 개의 SQL 문장을 UNION 오퍼레이션으로 처리하거나, CASE 기능을 구현하기 위해 DECODE 함수를 복잡하게 구현해야 하는 불편함이 있었다. 이런 불편 사항은 Oracle에서 표준 SQL에 포함된 CASE 기능과 FULL OUTER JOIN 기능을 추가함으로써 문제가 해결되었다.(참고로, Oracle DECODE 함수가 CASE 기능보다 장점도 있으므로 Oracle 사용자는 요구사항에 따라 DECODE나 CASE 함수를 선택할 수 있다.) 결과적으로 사용자 입장에서는 ANSI/ISO SQL의 새로운 기능들을 사용함으로써 보다 쉽게 데이터를 추출하거나 SQL 튜닝의 효과를 함께 얻을 수 있게 되었다. 대표적인 ANSI/ISO 표준 SQL의 기능은 다음 내용을 포함한다.

- STANDARD JOIN 기능 추가 (CROSS, OUTER JOIN 등 새로운 FROM 절 JOIN 기능들) - SCALAR SUBQUERY, TOP-N QUERY 등의 새로운 SUBQUERY 기능들 - ROLLUP, CUBE, GROUPING SETS 등의 새로운 리포팅 기능 - WINDOW FUNCTION 같은 새로운 개념의 분석 기능들

##### 가. 일반 집합 연산자

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_200.jpg)

현재 사용하는 SQL의 많은 기능이 관계형 데이터베이스의 이론을 수립한 E.F.Codd 박사의 논문에 언급이 되어 있다. 논문에 언급된 8가지 관계형 대수는 다시 각각 4개의 일반 집합 연산자와 순수 관계 연산자로 나눌 수 있으며, 관계형 데이터베이스 엔진 및 SQL의 기반 이론이 되었다. 일반 집합 연산자를 현재의 SQL과 비교하면,

1. UNION 연산은 UNION 기능으로, 2. INTERSECTION 연산은 INTERSECT 기능으로, 3. DIFFERENCE 연산은 EXCEPT(Oracle은 MINUS) 기능으로, 4. PRODUCT 연산은 CROSS JOIN 기능으로 구현되었다.

첫 번째, UNION 연산은 수학적 합집합을 제공하기 위해, 공통 교집합의 중복을 없애기 위한 사전 작업으로 시스템에 부하를 주는 정렬 작업이 발생한다. 이후 UNION ALL 기능이 추가되었는데, 특별한 요구 사항이 없다면 공통집합을 중복해서 그대로 보여 주기 때문에 정렬 작업이 일어나지 않는 장점을 가진다. 만일 UNION과 UNION ALL의 출력 결과가 같다면, 응답 속도 향상이나 자원 효율화 측면에서 데이터 정렬 작업이 발생하지 않는 UNION ALL을 사용하는 것을 권고한다. 두 번째, INTERSECTION은 수학의 교집합으로써 두 집합의 공통집합을 추출한다. 세 번째, DIFFERENCE는 수학의 차집합으로써 첫 번째 집합에서 두 번째 집합과의 공통집합을 제외한 부분이다. 대다수 벤더는 EXCEPT를, Oracle은 MINUS 용어를 사용한다. (SQL 표준에는 EXCEPT로 표시되어 있으며, 벤더에서 SQL 표준 기능을 구현할 때 다른 용어를 사용하는 것은 현실적으로 허용되고 있다.) 네 번째, PRODUCT의 경우는 CROSS(ANIS/ISO 표준) PRODUCT라고 불리는 곱집합으로, JOIN 조건이 없는 경우 생길 수 있는 모든 데이터의 조합을 말한다. 양쪽 집합의 M*N 건의 데이터 조합이 발생하며, CARTESIAN(수학자 이름) PRODUCT라고도 표현한다.

##### 나. 순수 관계 연산자

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_201.jpg)

순수 관계 연산자는 관계형 데이터베이스를 구현하기 위해 새롭게 만들어진 연산자이다. 순수 관계 연산자를 현재의 SQL 문장과 비교하면 다음과 같다.

5. SELECT 연산은 WHERE 절로 구현되었다. 6. PROJECT 연산은 SELECT 절로 구현되었다. 7. (NATURAL) JOIN 연산은 다양한 JOIN 기능으로 구현되었다. 8. DIVIDE 연산은 현재 사용되지 않는다.

다섯 번째, SELECT 연산은 SQL 문장에서는 WHERE 절의 조건절 기능으로 구현이 되었다. (SELECT 연산과 SELECT 절의 의미가 다름을 유의하자.) 여섯 번째, PROJECT 연산은 SQL 문장에서는 SELECT 절의 칼럼 선택 기능으로 구현이 되었다. 일곱 번째, JOIN 연산은 WHERE 절의 INNER JOIN 조건과 함께 FROM 절의 NATURAL JOIN, INNER JOIN, OUTER JOIN, USING 조건절, ON 조건절 등으로 가장 다양하게 발전하였다. 여덟 번째, DIVIDE 연산은 나눗셈과 비슷한 개념으로 왼쪽의 집합을 ‘XZ’로 나누었을 때, 즉 ‘XZ’를 모두 가지고 있는 ‘A’가 답이 되는 기능으로 현재 사용되지 않는다. 관계형 데이터베이스의 경우 요구사항 분석, 개념적 데이터 모델링, 논리적 데이터 모델링, 물리적 데이터 모델링 단계를 거치게 되는데, 이 단계에서 엔터티 확정 및 정규화 과정, 그리고 M:M (다대다) 관계를 분해하는 절차를 거치게 된다. 특히 정규화 과정의 경우 데이터 정합성과 데이터 저장 공간의 절약을 위해 엔터티를 최대한 분리하는 작업으로, 일반적으로 3차 정규형이나 보이스코드 정규형까지 진행하게 된다. 이런 정규화를 거치면 하나의 주제에 관련 있는 엔터티가 여러 개로 나누어지게 되고, 이 엔터티들이 주로 테이블이 되는데 이렇게 흩어진 데이터를 연결해서 원하는 데이터를 가져오는 작업이 바로 JOIN이라고 할 수 있다. 관계형 데이터베이스에 있어서 JOIN은 SQL의 가장 중요한 기능이므로 충분히 이해할 필요가 있다.

#### 2. FROM 절 JOIN 형태

ANSI/ISO SQL에서 표시하는 FROM 절의 JOIN 형태는 다음과 같다.

- INNER JOIN - NATURAL JOIN - USING 조건절 - ON 조건절 - CROSS JOIN - OUTER JOIN

ANSI/ISO SQL에서 규정한 JOIN 문법은 WHERE 절을 사용하던 기존 JOIN 방식과 차이가 있다. 사용자는 기존 WHERE 절의 검색 조건과 테이블 간의 JOIN 조건을 구분 없이 사용하던 방식을 그대로 사용할 수 있으면서, 추가된 선택 기능으로 테이블 간의 JOIN 조건을 FROM 절에서 명시적으로 정의할 수 있게 되었다. INNER JOIN은 WHERE 절에서부터 사용하던 JOIN의 DEFAULT 옵션으로 JOIN 조건에서 동일한 값이 있는 행만 반환한다. DEFAULT 옵션이므로 생략이 가능하지만, CROSS JOIN, OUTER JOIN과는 같이 사용할 수 없다. NATURAL JOIN은 INNER JOIN의 하위 개념으로 NATURAL JOIN은 두 테이블 간의 동일한 이름을 갖는 모든 칼럼들에 대해 EQUI(=) JOIN을 수행한다. NATURAL INNER JOIN이라고도 표시할 수 있으며, 결과는 NATURAL JOIN과 같다. 새로운 SQL JOIN 문장 중에서 가장 중요하게 기억해야 하는 문장은 ON 조건절을 사용하는 경우이다. 과거 WHERE 절에서 JOIN 조건과 데이터 검증 조건이 같이 사용되어 용도가 불분명한 경우가 발생할 수 있었는데, WHERE 절의 JOIN 조건을 FROM 절의 ON 조건절로 분리하여 표시함으로써 사용자가 이해하기 쉽도록 한다. ON 조건절의 경우 NATURAL JOIN처럼 JOIN 조건이 숨어 있지 않고, 명시적으로 JOIN 조건을 구분할 수 있고, NATURAL JOIN이나 USING 조건절처럼 칼럼명이 똑같아야 된다는 제약 없이 칼럼명이 상호 다르더라도 JOIN 조건으로 사용할 수 있으므로 앞으로 가장 많이 사용될 것으로 예상된다. 다만, FROM 절에 테이블이 많이 사용될 경우 다소 복잡하게 보여 가독성이 떨어지는 단점이 있다. 그런 측면에서 SQL Server의 경우 ON 조건절만 지원하고 NATURAL JOIN과 USING 조건절을 지원하지 않고 있는 것으로 보인다. 본 가이드는 ANSI/ISO SQL 기준에 NATURAL JOIN과 USING 조건절이 표시되어 있으므로 이 부분도 설명을 하도록 한다.

#### 3. INNER JOIN

INNER JOIN은 OUTER(외부) JOIN과 대비하여 내부 JOIN이라고 하며 JOIN 조건에서 동일한 값이 있는 행만 반환한다. INNER JOIN 표시는 그 동안 WHERE 절에서 사용하던 JOIN 조건을 FROM 절에서 정의하겠다는 표시이므로 USING 조건절이나 ON 조건절을 필수적으로 사용해야 한다.

[예제] 사원 번호와 사원 이름, 소속부서 코드와 소속부서 이름을 찾아본다.

[예제] WHERE 절 JOIN 조건 SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME FROM EMP, DEPT WHERE EMP.DEPTNO = DEPT.DEPTNO; 위 SQL과 아래 SQL은 같은 결과를 얻을 수 있다. FROM 절 JOIN 조건 SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME FROM EMP INNER JOIN DEPT ON EMP.DEPTNO = DEPT.DEPTNO; INNER는 JOIN의 디폴트 옵션으로 아래 SQL문과 같이 생략 가능하다. SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME FROM EMP JOIN DEPT ON EMP.DEPTNO = DEPT.DEPTNO;

[실행 결과] DEPTNO EMPNO ENAME DNAME ------ ----- ------ --------- 20 7369 SMITH RESEARCH 30 7499 ALLEN SALES 30 7521 WARD SALES 20 7566 JONES RESEARCH 30 7654 MARTIN SALES 30 7698 BLAKE SALES 10 7782 CLARK ACCOUNTING 20 7788 SCOTT RESEARCH 10 7839 KING ACCOUNTING 30 7844 TURNER SALES 20 7876 ADAMS RESEARCH 30 7900 JAMES SALES 20 7902 FORD RESEARCH 10 7934 MILLER ACCOUNTING 14개의 행이 선택되었다.

위에서 사용한 ON 조건절에 대해서는 뒤에서 추가 설명하도록 한다.

#### 4. NATURAL JOIN

NATURAL JOIN은 두 테이블 간의 동일한 이름을 갖는 모든 칼럼들에 대해 EQUI(=) JOIN을 수행한다. NATURAL JOIN이 명시되면, 추가로 USING 조건절, ON 조건절, WHERE 절에서 JOIN 조건을 정의할 수 없다. 그리고, SQL Server에서는 지원하지 않는 기능이다.

[예제] 사원 번호와 사원 이름, 소속부서 코드와 소속부서 이름을 찾아본다.

[예제] SELECT DEPTNO, EMPNO, ENAME, DNAME FROM EMP NATURAL JOIN DEPT;

[실행 결과] DEPTNO EMPNO ENAME DNAME ------ ------ ------ ------ 20 7369 SMITH RESEARCH 30 7499 ALLEN SALES 30 7521 WARD SALES 20 7566 JONES RESEARCH 30 7654 MARTIN SALES 30 7698 BLAKE SALES 10 7782 CLARK ACCOUNTING 20 7788 SCOTT RESEARCH 10 7839 KING ACCOUNTING 30 7844 TURNER SALES 20 7876 ADAMS RESEARCH 30 7900 JAMES SALES 20 7902 FORD RESEARCH 10 7934 MILLER ACCOUNTING 14개의 행이 선택되었다.

위 SQL은 별도의 JOIN 칼럼을 지정하지 않았지만, 두 개의 테이블에서 DEPTNO라한 것이다. JOIN에 사용된 칼럼들은 같은 데이터 유형이어야 하며, ALIAS나 테이블 명과 같은 접두사를 붙일 수 없다.

[예제] SELECT EMP.DEPTNO, EMPNO, ENAME, DNAME FROM EMP NATURAL JOIN DEPT; ERROR: NATURAL JOIN에 사용된 열은 식별자를 가질 수 없음

NATURAL JOIN은 JOIN이 되는 테이블의 데이터 성격(도메인)과 칼럼명 등이 동일해야 하는 제약 조건이 있다. 간혹 모델링 상의 부주의로 인해 동일한 칼럼명이더라도 다른 용도의 데이터를 저장하는 경우도 있으므로 주의해서 사용해야 한다.

[예제] 아래 '*' 와일드카드처럼 별도의 칼럼 순서를 지정하지 않으면 NATURAL JOIN의 기준이 되는 칼럼 들이 다른 칼럼보다 먼저 출력된다. (ex: DEPTNO가 첫 번째 칼럼이 된다.) 이때 NATURAL JOIN은 JOIN에 사용된 같은 이름의 칼럼을 하나로 처리한다.

[예제] SELECT * FROM EMP NATURAL JOIN DEPT;

[실행 결과] DEPTNO EMPNO ENAME JOB MGR HIREDATE SAL COMM DNAME LOC ----- ----- ----- -------- --- -------- ---- ---- --------- ------ 20 7369 SMITH CLERK 7902 1980-12-17 800 RESEARCH DALLAS 30 7499 ALLEN SALESMAN 7698 1981-02-20 1600 300 SALES CHICAGO 30 7521 WARD SALESMAN 7698 1981-02-22 1250 500 SALES CHICAGO 20 7566 JONES MANAGER 7839 1981-04-02 2975 RESEARCH DALLAS 30 7654 MARTIN SALESMAN 7698 1981-09-28 1250 1400 SALES CHICAGO 30 7698 BLAKE MANAGER 7839 1981-05-01 2850 SALES CHICAGO 10 7782 CLARK MANAGER 7839 1981-06-09 2450 ACCOUNTING NEW YORK 20 7788 SCOTT ANALYST 7566 1987-07-13 3000 RESEARCH DALLAS 10 7839 KING PRESIDENT 1981-11-17 5000 ACCOUNTING NEW YORK 30 7844 TURNER SALESMAN 7698 1981-09-08 1500 SALES CHICAGO 20 7876 ADAMS CLERK 7788 1987-07-13 1100 RESEARCH DALLAS 30 7900 JAMES CLERK 7698 1981-12-03 950 0 SALES CHICAGO 20 7902 FORD ANALYST 7566 1981-12-03 3000 RESEARCH DALLAS 10 7934 MILLER CLERK 7782 1982-01-23 1300 ACCOUNTING NEW YORK 14개의 행이 선택되었다.

[예제] 반면, INNER JOIN의 경우 첫 번째 테이블, 두 번째 테이블의 칼럼 순서대로 데이터가 출력된다. 이때 NATURAL JOIN은 JOIN에 사용된 같은 이름의 칼럼을 하나로 처리하지만, INNER JOIN은 별개의 칼럼으로 표시한다.

[예제] SELECT * FROM EMP INNER JOIN DEPT ON EMP.DEPTNO = DEPT.DEPTNO;

[실행 결과] EMPNO ENAME JOB MGR HIREDATE SAL COMM DEPTNO DEPTNO DNAME LOC ---- ----- ------ --- ------- --- ---- ----- ----- -------- ----- 7369 SMITH CLERK 7902 1980-12-17 800 20 20 RESEARCH DALLAS 7499 ALLEN SALESMAN 7698 1981-02-20 1600 300 30 30 SALES CHICAGO 7521 WARD SALESMAN 7698 1981-02-22 1250 500 30 30 SALES CHICAGO 7566 JONES MANAGER 7839 1981-04-02 2975 20 20 RESEARCH DALLAS 7654 MARTIN SALESMAN 7698 1981-09-28 1250 1400 30 30 SALES CHICAGO 7698 BLAKE MANAGER 7839 1981-05-01 2850 30 30 SALES CHICAGO 7782 CLARK MANAGER 7839 1981-06-09 2450 10 10 ACCOUNTING NEW YORK 7788 SCOTT ANALYST 7566 1987-07-13 3000 20 20 RESEARCH DALLAS 7839 KING PRESIDENT 1981-11-17 5000 10 10 ACCOUNTING NEW YORK 7844 TURNER SALESMAN 7698 1981-09-08 1500 0 30 30 SALES CHICAGO 7876 ADAMS CLERK 7788 1987-07-13 1100 20 20 RESEARCH DALLAS 7900 JAMES CLERK 7698 1981-12-03 950 30 30 SALES CHICAGO 7902 FORD ANALYST 7566 1981-12-03 3000 20 20 RESEARCH DALLAS 7934 MILLER CLERK 7782 1982-01-23 1300 10 10 ACCOUNTING NEW YORK 14개의 행이 선택되었다.

[예제] NATURAL JOIN과 INNER JOIN의 차이를 자세히 설명하기 위해 DEPT_TEMP 테이블을 임시로 만든다.

[예제] Oracle CREATE TABLE DEPT_TEMP AS SELECT * FROM DEPT;

[예제] SQL Server SELECT * INTO DEPT_TEMP FROM DEPT;

[예제] UPDATE DEPT_TEMP SET DNAME = 'R&D' WHERE DNAME = 'RESEARCH'; UPDATE DEPT_TEMP SET DNAME = 'MARKETING' WHERE DNAME = 'SALES'; SELECT * FROM DEPT_TEMP;

[실행 결과] DEPTNO DNAME LOC -------- ---------- --------- 10 ACCOUNTING NEW YORK 20 R&D DALLAS 30 MARKETING CHICAGO 40 OPERATIONS BOSTON 4개의 행이 선택되었다.

부서번호 20과 30의 DNAME이 'R&D'와 'MARKETING'으로 변경된 것을 확인할 수 있다.

[예제] 세 개의 칼럼명이 모두 같은 DEPT와 DEPT_TEMP 테이블을 NATURAL [INNER] JOIN으로 수행한다.

[예제] SELECT * FROM DEPT NATURAL INNER JOIN DEPT_TEMP; INNER는 DEFAULT 옵션으로 아래와 같이 생략? 수 있다. SELECT * FROM DEPT NATURAL JOIN DEPT_TEMP;

[실행 결과] DEPTNO DNAME LOC ------ ---------- ---------- 10 ACCOUNTING NEW YORK 40 OPERATIONS BOSTON 2개의 행이 선택되었다.

위 SQL의 경우 DNAME의 내용이 바뀐 부서번호 20, 30의 데이터는 실행 결과에서 제외된 것을 알 수 있다.

[예제] 다음에는 같은 조건이지만 출력 칼럼에서 차이가 나는 일반적인 INNER JOIN을 수행한다.

[예제] SELECT * FROM DEPT JOIN DEPT_TEMP ON DEPT.DEPTNO = DEPT_TEMP.DEPTNO AND DEPT.DNAME = DEPT_TEMP.DNAME AND DEPT.LOC = DEPT_TEMP.LOC; 위 SQL과 아래 SQL은 같은 결과를 얻을 수 있다. SELECT * FROM DEPT, DEPT_TEMP WHERE DEPT.DEPTNO = DEPT_TEMP.DEPTNO AND DEPT.DNAME = DEPT_TEMP.DNAME AND DEPT.LOC = DEPT_TEMP.LOC;

[실행 결과] DEPTNO DNAME LOC DEPTNO DNAME LOC ------ ---------- -------- ------ ---------- ------ 10 ACCOUNTING NEW YORK 10 ACCOUNTING NEW YORK 40 OPERATIONS BOSTON 40 OPERATIONS BOSTON 2개의 행이 선택되었다.

위 SQL의 경우 DNAME의 내용이 바뀐 부서번호 20, 30의 경우는 결과에서 제외된 것을 알 수 있다. 차이가 나는 부분은 NATURAL JOIN은 JOIN에 사용된 같은 이름의 칼럼을 하나로 처리하지만, INNER JOIN의 경우는 2개의 칼럼으로 표시된다.

#### 5. USING 조건절

NATURAL JOIN에서는 모든 일치되는 칼럼들에 대해 JOIN이 이루어지지만, FROM 절의 USING 조건절을 이용하면 같은 이름을 가진 칼럼들 중에서 원하는 칼럼에 대해서만 선택적으로 EQUI JOIN을 할 수가 있다. 다만, 이 기능은 SQL Server에서는 지원하지 않는다.

[예제] 세 개의 칼럼명이 모두 같은 DEPT와 DEPT_TEMP 테이블을 DEPTNO 칼럼을 이용한 [INNER] JOIN의 USING 조건절로 수행한다.

[예제] SELECT * FROM DEPT JOIN DEPT_TEMP USING (DEPTNO);

[실행 결과] DEPTNO DNAME LOC DNAME LOC ------ ---------- --------- ---------- --------- 10 ACCOUNTING NEW YORK ACCOUNTING NEW YORK 20 RESEARCH DALLAS R&D DALLAS 30 SALES CHICAGO MARKETING CHICAGO 40 OPERATIONS BOSTON OPERATIONS BOSTON 4개의 행이 선택되었다.

위 SQL의 '*' 와일드카드처럼 별도의 칼럼 순서를 지정하지 않으면 USING 조건절의 기준이 되는 칼럼이 다른 칼럼보다 먼저 출력된다. (ex: DEPTNO가 첫 번째 칼럼이 된다.) 이때 USING JOIN은 JOIN에 사용된 같은 이름의 칼럼을 하나로 처리한다.

[예제] USING 조건절을 이용한 EQUI JOIN에서도 NATURAL JOIN과 마찬가지로 JOIN 칼럼에 대해서는 ALIAS나 테이블 이름과 같은 접두사를 붙일 수 없다. (DEPT.DEPTNO → DEPTNO)

[예제] 잘못된 사례: SELECT DEPT.DEPTNO, DEPT.DNAME, DEPT.LOC, DEPT_TEMP.DNAME, DEPT_TEMP.LOC FROM DEPT JOIN DEPT_TEMP USING (DEPTNO); ERROR: USING 절의 열 부분은 식별자를 가질 수 없음 바른 사례: SELECT DEPTNO, DEPT.DNAME, DEPT.LOC, DEPT_TEMP.DNAME, DEPT_TEMP.LOC FROM DEPT JOIN DEPT_TEMP USING (DEPTNO);

[실행 결과] DEPTNO DNAME LOC DNAME LOC ------- --------- --------- ----------- -------- 10 ACCOUNTING NEW YORK ACCOUNTING NEW YORK 20 RESEARCH DALLAS R&D DALLAS 30 SALES CHICAGO MARKETING CHICAGO 40 OPERATIONS BOSTON OPERATIONS BOSTON 4개의 행이 선택되었다.

[예제] 이번에는 DEPT와 DEPT_TEMP 테이블의 일부 데이터 내용이 변경되었던 DNAME 칼럼을 조인 조건으로 [INNER] JOIN의 USING 조건절을 수행한다.

[예제] SELECT * FROM DEPT JOIN DEPT_TEMP USING (DNAME);

[실행 결과] DNAME DEPTNO LOC DEPTNO LOC ---------- ------ --------- ------- --------- ACCOUNTING 10 NEW YORK 10 NEW YORK OPERATIONS 40 BOSTON 40 BOSTON 2개의 행이 선택되었다.

위 SQL의 경우 DNAME의 내용이 바뀐 부서번호 20, 30의 경우는 결과에서 제외된 것을 알 수 있다. 그리고 USING에 사용된 DNAME이 첫 번째 칼럼으로 출력된 것과 함께, JOIN 조건에 참여하지 않은 DEPTNO와 LOC가 2개의 칼럼으로 표시된 것을 알 수 있다.

[예제] 이번에는 세 개의 칼럼명이 모두 같은 DEPT와 DEPT_TEMP 테이블을 LOC와 DEPTNO 2개 칼럼을 이용한 [INNER] JOIN의 USING 조건절로 수행한다.

[예제] SELECT * FROM DEPT JOIN DEPT_TEMP USING (LOC, DEPTNO);

[실행 결과] LOC DEPTNO DNAME DNAME -------- ------ ---------- ---------- NEW YORK 10 ACCOUNTING ACCOUNTING DALLAS 20 RESEARCH R&D CHICAGO 30 SALES MARKETING BOSTON 40 OPERATIONS OPERATIONS 4개의 행이 선택되었다.

USING에 사용된 LOC, DEPTNO가 첫 번째, 두 번째 칼럼으로 출력되고, JOIN 조건에 참여하지 않은 DNAME 칼럼은 2개의 칼럼으로 표시된 것을 알 수 있다.

[예제] 이번에는 DEPTNO, DNAME 2개의 칼럼을 이용한 [INNER] JOIN의 USING 조건절로 수행한다.

[예제] SELECT * FROM DEPT JOIN DEPT_TEMP USING (DEPTNO, DNAME);

[실행 결과] DEPTNO DNAME LOC LOC ------ ---------- -------- -------- 10 ACCOUNTING NEW YORK NEW YORK 40 OPERATIONS BOSTON BOSTON 2개의 행이 선택되었다.

위 SQL의 경우 DNAME의 내용이 바뀐 부서번호 20, 30의 경우는 결과에서 제외된 것을 알 수 있다. 그리고 USING에 사용된 DEPTNO, DNAME이 첫 번째, 두 번째 칼럼으로 출력된 것과 함께, JOIN 조건에 참여하지 않은 LOC가 2개의 칼럼으로 표시된 것을 알 수 있다

#### 6. ON 조건절

JOIN 서술부(ON 조건절)와 비 JOIN 서술부(WHERE 조건절)를 분리하여 이해가 쉬우며, 칼럼명이 다르더라도 JOIN 조건을 사용할 수 있는 장점이 있다.

[예제] 사원 테이블과 부서 테이블에서 사원 번호와 사원 이름, 소속부서 코드, 소속부서 이름을 출력한다.

[예제] SELECT E.EMPNO, E.ENAME, E.DEPTNO, D.DNAME FROM EMP E JOIN DEPT D ON (E.DEPTNO = D.DEPTNO);

[실행 결과] EMPNO ENAME DEPTNO DNAME ----- ------- ------ ----------- 7369 SMITH 20 RESEARCH 7499 ALLEN 30 SALES 7521 WARD 30 SALES 7566 JONES 20 RESEARCH 7654 MARTIN 30 SALES 7698 BLAKE 30 SALES 7782 CLARK 10 ACCOUNTING 7788 SCOTT 20 RESEARCH 7839 KING 10 ACCOUNTING 7844 TURNER 30 SALES 7876 ADAMS 20 RESEARCH 7900 JAMES 30 SALES 7902 FORD 20 RESEARCH 7934 MILLER 10 ACCOUNTING 14개의 행이 선택되었다.

NATURAL JOIN의 JOIN 조건은 기본적으로 같은 이름을 가진 모든 칼럼들에 대한 동등 조건이지만, 임의의 JOIN 조건을 지정하거나, 이름이 다른 칼럼명을 JOIN 조건으로 사용하거나, JOIN 칼럼을 명시하기 위해서는 ON 조건절을 사용한다. ON 조건절에 사용된 괄호는 옵션 사항이다. USING 조건절을 이용한 JOIN에서는 JOIN 칼럼에 대해서 ALIAS나 테이블 명과 같은 접두사를 사용하면 SYNTAX 에러가 발생하지만, 반대로 ON 조건절을 사용한 JOIN의 경우는 ALIAS나 테이블 명과 같은 접두사를 사용하여 SELECT에 사용되는 칼럼을 논리적으로 명확하게 지정해주어야 한다. (DEPTNO → E.DEPTNO) ON 조건절은 WHERE 절의 JOIN 조건과 같은 기능을 하면서도, 명시적으로 JOIN의 조건을 구분할 수 있으므로 가장 많이 사용될 것으로 예상된다. 다만, FROM 절에 테이블이 많이 사용될 경우 다소 복잡하게 보여 가독성이 떨어지는 단점이 있다.

##### 가. WHERE 절과의 혼용

[예제] ON 조건절과 WHERE 검색 조건은 충돌 없이 사용할 수 있다. 부서코드 30인 부서의 소속 사원 이름 및 소속 부서 코드, 부서 코드, 부서 이름을 찾아본다.

[예제] SELECT E.ENAME, E.DEPTNO, D.DEPTNO, D.DNAME FROM EMP E JOIN DEPT D ON (E.DEPTNO = D.DEPTNO) WHERE E.DEPTNO = 30;

[실행 결과] ENAME DEPTNO DEPTNO DNAME ------- ------ ------ ------ ALLEN 30 30 SALES WARD 30 30 SALES MARTIN 30 30 SALES BLAKE 30 30 SALES TURNER 30 30 SALES JAMES 30 30 SALES 6개의 행이 선택되었다.

##### 나. ON 조건절 + 데이터 검증 조건 추가

ON 조건절에 JOIN 조건 외에도 데이터 검색 조건을 추가할 수는 있으나, 검색 조건 목적인 경우는 WHERE 절을 사용할 것을 권고한다. (다만, 아우터 조인에서 조인의 대상을 제한하기 위한 목적으로 사용되는 추가 조건의 경우는 ON 절에 표기되어야 한다.)

[예제] 매니저 사원번호가 7698번인 사원들의 이름 및 소속 부서 코드, 부서 이름을 찾아본다.

[예제] SELECT E.ENAME, E.MGR, D.DEPTNO, D.DNAME FROM EMP E JOIN DEPT D ON (E.DEPTNO = D.DEPTNO AND E.MGR = 7698); 위 SQL과 아래 SQL은 같은 결과를 얻을 수 있다. SELECT E.ENAME, E.MGR, D.DEPTNO, D.DNAME FROM EMP E JOIN DEPT D ON (E.DEPTNO = D.DEPTNO) WHERE E.MGR = 7698;

[실행 결과] ENAME MGR DEPTNO DNAME ------- ---- ------ ------ ALLEN 7698 30 SALES WARD 7698 30 SALES MARTIN 7698 30 SALES TURNER 7698 30 SALES JAMES 7698 30 SALES 5개의 행이 선택되었다.

##### 다. ON 조건절 예제

[예제] 팀과 스타디움 테이블을 스타디움ID로 JOIN하여 팀이름, 스타디움ID, 스타디움 이름을 찾아본다.

[예제] SELECT TEAM_NAME, TEAM.STADIUM_ID, STADIUM_NAME FROM TEAM JOIN STADIUM ON TEAM.STADIUM_ID = STADIUM.STADIUM_ID ORDER BY STADIUM_ID;; 위 SQL은 STADIUM_ID라는 공통된 칼럼이 있기 때문에 아래처럼 USING 조건절로 구현할 수도 있다. SELECT TEAM_NAME, STADIUM_ID, STADIUM_NAME FROM TEAM JOIN STADIUM USING (STADIUM_ID) ORDER BY STADIUM_ID; 위 SQL은 고전적인 방식인 WHERE 절의 INNER JOIN으로 구현할 수도 있다. SELECT TEAM_NAME, TEAM.STADIUM_ID, STADIUM_NAME FROM TEAM, STADIUM WHERE TEAM.STADIUM_ID = STADIUM.STADIUM_ID ORDER BY STADIUM_ID

[실행 결과] TEAM_NAME STADIUM_ID STADIUM_NAME ------------- --------- ------------- 광주상무 A02 광주월드컵경기장 강원FC A03 강릉종합경기장 제주유나이티드FC A04 제주월드컵경기장 대구FC A05 대구월드컵경기장 유나이티드 B01 인천월드컵경기장 일화천마 B02 성남종합운동장 삼성블루윙즈 B04 수원월드컵경기장 FC서울 B05 서울월드컵경기장 아이파크 C02 부산아시아드경기장 울산현대 C04 울산문수경기장 경남FC C05 창원종합운동장 스틸러스 C06 포항스틸야드 드래곤즈 D01 광양전용경기장 시티즌 D02 대전월드컵경기장 15개의 행이 선택되었다.

[예제] 팀과 스타디움 테이블을 팀ID로 JOIN하여 팀이름, 팀ID, 스타디움 이름을 찾아본다. STADIUM에는 팀ID가 HOMETEAM_ID라는 칼럼으로 표시되어 있다.

[예제] SELECT TEAM_NAME, TEAM_ID, STADIUM_NAME FROM TEAM JOIN STADIUM ON TEAM.TEAM_ID = STADIUM.HOMETEAM_ID ORDER BY TEAM_ID; 위 SQL은 고전적인 방식인 WHERE 절의 INNER JOIN으로 구현할 수도 있다. SELECT TEAM_NAME, TEAM_ID, STADIUM_NAME FROM TEAM, STADIUM WHERE TEAM.TEAM_ID = STADIUM.HOMETEAM_ID ORDER BY TEAM_ID; 위 SQL은 TEAM_ID와 HOMETEAM_ID라는 다른 이름의 칼럼을 사용하기 때문에 USING 조건절을 사용할 수는 없다.

[실행 결과] TEAM_NAME TEAM_ID STADIUM_NAME ----------- ------- ------------- 울산현대 K01 울산문수경기장 삼성블루윙즈 K02 수원월드컵경기장 스틸러스 K03 포항스틸야드 유나이티드 K04 인천월드컵경기장 현대모터스 K05 전주월드컵경기장 아이파크 K06 부산아시아드경기장 드래곤즈 K07 광양전용경기장 일화천마 K08 성남종합운동장 FC서울 K09 서울월드컵경기장 시티즌 K10 대전월드컵경기장 경남FC K11 창원종합운동장 광주상무 K12 광주월드컵경기장 강원FC K13 강릉종합경기장 제주유나이티드FC K14 제주월드컵경기장 15개의 행이 선택되었다.

##### 라. 다중 테이블 JOIN

[예제] 사원과 DEPT 테이블의 소속 부서명, DEPT_TEMP 테이블의 바뀐 부서명 정보를 출력한다.

[예제] SELECT E.EMPNO, D.DEPTNO, D.DNAME, T.DNAME New_DNAME FROM EMP E JOIN DEPT D ON (E.DEPTNO = D.DEPTNO) JOIN DEPT_TEMP T ON (E.DEPTNO = T.DEPTNO); 위 SQL은 고전적인 방식인 WHERE 절의 INNER JOIN으로 구현할 수도 있다. SELECT E.EMPNO, D.DEPTNO, D.DNAME, T.DNAME New_DNAME FROM EMP E, DEPT D, DEPT_TEMP T WHERE E.DEPTNO = D.DEPTNO AND E.DEPTNO = T.DEPTNO;

[실행 결과] EMPNO DEPTNO DNAME NEW_DNAME ------ ------ --------- ----------- 7369 20 RESEARCH R&D 7499 30 SALES MARKETING 7521 30 SALES MARKETING 7566 20 RESEARCH R&D 7654 30 SALES MARKETING 7698 30 SALES MARKETING 7782 10 ACCOUNTING ACCOUNTING 7788 20 RESEARCH R&D 7839 10 ACCOUNTING ACCOUNTING 7844 30 SALES MARKETING 7876 20 RESEARCH R&D 7900 30 SALES MARKETING 7902 20 RESEARCH R&D 7934 10 ACCOUNTING ACCOUNTING 14개의 행이 선택되었다.

[예제] GK 포지션의 선수별 연고지명, 팀명, 구장명을 출력한다.

[예제] SELECT P.PLAYER_NAME 선수명, P.POSITION 포지션, T.REGION_NAME 연고지명, T.TEAM_NAME 팀명, S.STADIUM_NAME 구장명 FROM PLAYER P JOIN TEAM T ON P.TEAM_ID = T.TEAM_ID JOIN STADIUM S ON T.STADIUM_ID = S.STADIUM_ID WHERE P.POSITION = 'GK' ORDER BY 선수명; 위 SQL은 고전적인 방식인 WHERE 절의 INNER JOIN으로 구현할 수도 있다. SELECT P.PLAYER_NAME 선수명, P.POSITION 포지션, T.REGION_NAME 연고지명, T.TEAM_NAME 팀명, S.STADIUM_NAME 구장명 FROM PLAYER P, TEAM T, STADIUM S WHERE P.TEAM_ID = T.TEAM_ID AND T.STADIUM_ID = S.STADIUM_ID AND P.POSITION = 'GK' ORDER BY 선수명;

[실행 결과] 선수명 포지션 연고지명 팀명 구장명 ----- ---- ------ -------- ---------- 강성일 GK 대전 시티즌 대전월드컵경기장 권정혁 GK 울산 울산현대 울산문수경기장 권찬수 GK 성남 일화천마 성남종합운동장 김대희 GK 포항 스틸러스 포항스틸야드 김승준 GK 대전 시티즌 대전월드컵경기장 김용발 GK 전북 현대모터스 전주월드컵경기장 김운재 GK 수원 삼성블루윙즈 수원월드컵경기장 김정래 GK 전남 드래곤즈 광양전용경기장 김준호 GK 포항 스틸러스 포항스틸야드 김창민 GK 전북 현대모터스 전주월드컵경기장 김충호 GK 인천 유나이티드 인천월드컵경기장 남현우 GK 인천 유나이티드 인천월드컵경기장 박유석 GK 부산 아이파크 부산아시아드경기장 43개의 행이 선택되었다.

[예제] 홈팀이 3점 이상 차이로 승리한 경기의 경기장 이름, 경기 일정, 홈팀 이름과 원정팀 이름 정보를 출력한다.

[예제] SELECT ST.STADIUM_NAME, SC.STADIUM_ID, SCHE_DATE, HT.TEAM_NAME, AT.TEAM_NAME, HOME_SCORE, AWAY_SCORE FROM SCHEDULE SC JOIN STADIUM ST ON SC.STADIUM_ID = ST.STADIUM_ID JOIN TEAM HT ON SC.HOMETEAM_ID = HT.TEAM_ID JOIN TEAM AT ON SC.AWAYTEAM_ID = AT.TEAM_ID WHERE HOME_SCORE > = AWAY_SCORE +3; 위 SQL은 고전적인 방식인 WHERE 절의 INNER JOIN으로 구현할 수도 있다. SELECT ST.STADIUM_NAME, SC.STADIUM_ID, SCHE_DATE, HT.TEAM_NAME, AT.TEAM_NAME, HOME_SCORE, AWAY_SCORE FROM SCHEDULE SC, STADIUM ST, TEAM HT, TEAM AT WHERE HOME_SCORE> = AWAY_SCORE +3 AND SC.STADIUM_ID = ST.STADIUM_ID AND SC.HOMETEAM_ID = HT.TEAM_ID AND SC.AWAYTEAM_ID = AT.TEAM_ID; FROM 절에 4개의 테이블이 JOIN에 참여하였으며, HOME TEAM과 AWAY TEAM의 팀 이름을 구하기 위해 TEAM 테이블을 HT와 AT 두 개의 ALIAS로 구분하였다.

[실행 결과] STADIUM_NAME STADIUM_ID SCHE_DATE TEAM_NAME TEAM_NAME HOME_SCORE AWAY_SCORE ------------ --------- -------- --------- --------- --------- --------- 서울월드컵경기장 B05 20120714 FC서울 삼성블루윙즈 3 0 부산아시아드경기장 C02 20120727 아이파크 시티즌 3 0 울산문수경기장 C04 20120803 울산현대 스틸러스 3 0 성남종합운동장 B02 20120317 일화천마 유나이티드 6 0 창원종합운동장 C05 20120427 경남FC 아이파크 5 2 5개의 행이 선택되었다.

#### 7. CROSS JOIN

CROSS JOIN은 E.F.CODD 박사가 언급한 일반 집합 연산자의 PRODUCT의 개념으로 테이블 간 JOIN 조건이 없는 경우 생길 수 있는 모든 데이터의 조합을 말한다. 두 개의 테이블에 대한 CARTESIAN PRODUCT 또는 CROSS PRODUCT와 같은 표현으로, 결과는 양쪽 집합의 M*N 건의 데이터 조합이 발생한다. (아래 56건의 데이터는 EMP 14건 * DEPT 4건의 데이터 조합 건수이다.)

[예제] 사원 번호와 사원 이름, 소속부서 코드와 소속부서 이름을 찾아본다.

[예제] SELECT ENAME, DNAME FROM EMP CROSS JOIN DEPT ORDER BY ENAME;

[실행 결과] ENAME DNAME -------- --------- ADAMS SALES ADAMS RESEARCH ADAMS OPERATIONS ADAMS ACCOUNTING ALLEN OPERATIONS ALLEN RESEARCH ALLEN ACCOUNTING ALLEN SALES BLAKE SALES BLAKE OPERATIONS BLAKE RESEARCH BLAKE ACCOUNTING CLARK SALES CLARK RESEARCH CLARK OPERATIONS CLARK ACCOUNTING 56개의 행이 선택되었다.

[예제] NATURAL JOIN의 경우 WHERE 절에서 JOIN 조건을 추가할 수 없지만, CROSS JOIN의 경우 WHERE 절에 JOIN 조건을 추가할 수 있다. 그러나, 이 경우는 CROSS JOIN이 아니라 INNER JOIN과 같은 결과를 얻기 때문에 CROSS JOIN을 사용하는 의미가 없어지므로 권고하지 않는다.

[예제] SELECT ENAME, DNAME FROM EMP CROSS JOIN DEPT WHERE EMP.DEPTNO = DEPT.DEPTNO; 위 SQL과 아래 SQL은 같은 결과를 얻을 수 있다. SELECT ENAME, DNAME FROM EMP INNER JOIN DEPT WHERE EMP.DEPTNO = DEPT.DEPTNO;

[실행 결과] ENAME DNAME ------- --------- SMITH RESEARCH ALLEN SALES WARD SALES JONES RESEARCH MARTIN SALES BLAKE SALES CLARK ACCOUNTING SCOTT RESEARCH KING ACCOUNTING TURNER SALES ADAMS RESEARCH JAMES SALES FORD RESEARCH MILLER ACCOUNTING 14개의 행이 선택되었다.

정상적인 데이터 모델이라면 CROSS PRODUCT가 필요한 경우는 많지 않지만, 간혹 튜닝이나 리포트를 작성하기 위해 고의적으로 사용하는 경우가 있을 수 있다. 그리고 데이터웨어하우스의 개별 DIMENSION(차원)을 FACT(사실) 칼럼과 JOIN하기 전에 모든 DIMENSION의 CROSS PRODUCT를 먼저 구할 때 유용하게 사용할 수 있다.

#### 8. OUTER JOIN

INNER(내부) JOIN과 대비하여 OUTER(외부) JOIN이라고 불리며, JOIN 조건에서 동일한 값이 없는 행도 반환할 때 사용할 수 있다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_202.jpg)

[그림 Ⅱ-2-3]은 TAB1 테이블이 TAB2 테이블을 JOIN 하되, TAB2의 JOIN 데이터가 있는 경우는 TAB2의 데이터를 함께 출력하고, TAB2의 JOIN 데이터가 없는 경우에도 TAB1의 모든 데이터를 표시하고 싶은 경우이다. TAB1의 모든 값에 대해 TAB2의 데이터가 반드시 존재한다는 보장이 없는 경우 OUTER JOIN을 사용하여 해결이 가능하다. 과거 OUTER JOIN을 위해 Oracle은 JOIN 칼럼 뒤에 ‘(+)’를 표시하였고, Sybase는 비교 연산자의 앞이나 뒤에 ‘(+)’를 표시했었는데, JOIN 조건과 WHERE 절 검색 조건이 불명확한 단점, IN이나 OR 연산자 사용시 에러 발생, ‘(+)’ 표시가 누락된 칼럼 존재시 OUTER JOIN 오류 발생, FULL OUTER JOIN 미지원 등 불편함이 많았다. STANDARD JOIN을 사용함으로써 OUTER JOIN의 많은 문제점을 해결할 수 있고, 대부분의 관계형 DBMS 간에 호환성을 확보할 수 있으므로 명시적인 OUTER JOIN을 사용할 것을 적극적으로 권장한다. 추가로 OUTER JOIN 역시 JOIN 조건을 FROM 절에서 정의하겠다는 표시이므로 USING 조건절이나 ON 조건절을 필수적으로 사용해야 한다. 그리고, LEFT/RIGHT OUTER JOIN의 경우에는 기준이 되는 테이블이 조인 수행시 무조건 드라이빙 테이블이 된다. 옵티마이저는 이 원칙에 위배되는 다른 실행계획을 고려하지 않는다.

##### 가. LEFT OUTER JOIN

조인 수행시 먼저 표기된 좌측 테이블에 해당하는 데이터를 먼저 읽은 후, 나중 표기된 우측 테이블에서 JOIN 대상 데이터를 읽어 온다. 즉, Table A와 B가 있을 때(Table 'A'가 기준이 됨), A와 B를 비교해서 B의 JOIN 칼럼에서 같은 값이 있을 때 그 해당 데이터를 가져오고, B의 JOIN 칼럼에서 같은 값이 없는 경우에는 B 테이블에서 가져오는 칼럼들은 NULL 값으로 채운다. 그리고 LEFT JOIN으로 OUTER 키워드를 생략해서 사용할 수 있다.

[예제] STADIUM에 등록된 운동장 중에는 홈팀이 없는 경기장도 있다. STADIUM과 TEAM을 JOIN 하되 홈팀이 없는 경기장의 정보도 같이 출력하도록 한다.

[예제] SELECT STADIUM_NAME, STADIUM.STADIUM_ID, SEAT_COUNT, HOMETEAM_ID, TEAM_NAME FROM STADIUM LEFT OUTER JOIN TEAM ON STADIUM.HOMETEAM_ID = TEAM.TEAM_ID ORDER BY HOMETEAM_ID; OUTER는 생략 가능한 키워드이므로 아래 SQL은 같은 결과를 얻을 수 있다. SELECT STADIUM_NAME, STADIUM.STADIUM_ID, SEAT_COUNT, HOMETEAM_ID, TEAM_NAME FROM STADIUM LEFT JOIN TEAM ON STADIUM.HOMETEAM_ID = TEAM.TEAM_ID ORDER BY HOMETEAM_ID;

[실행 결과] STADIUM_NAME STADIUM_ID SEAT_COUNT HOMETEAM_ID TEAM_NAME ------------ --------- ---------- ----------- ---------- 울산문수경기장 C04 46102 K01 울산현대 수원월드컵경기장 B04 50000 K02 삼성블루윙즈 포항스틸야드 C06 25000 K03 스틸러스 인천월드컵경기장 B01 35000 K04 유나이티드 전주월드컵경기장 D03 28000 K05 현대모터스 부산아시아드경기장 C02 30000 K06 아이파크 광양전용경기장 D01 20009 K07 드래곤즈 성남종합운동장 B02 27000 K08 일화천마 서울월드컵경기장 B05 66806 K09 FC서울 대전월드컵경기장 D02 41000 K10 시티즌 창원종합운동장 C05 27085 K11 경남FC 광주월드컵경기장 A02 40245 K12 광주상무 강릉종합경기장 A03 33000 K13 강원FC 제주월드컵경기장 A04 42256 K14 제주유나이티드FC 대구월드컵경기장 A05 66422 K15 대구FC 안양경기장 F05 20000 마산경기장 F04 20000 일산경기장 F03 20000 부산시민경기장 F02 30000 대구시민경기장 F01 30000 20개의 행이 선택되었다.

INNER JOIN이라면 홈팀이 배정된 15개의 경기장만 출력 되었겠지만, LEFT OUTER JOIN을 사용하였기 때문에 홈팀이 없는 대구시민경기장, 부산시민경기장, 일산경기장, 마산경기장, 안양경기장의 정보까지 추가로 출력되었다.

##### 나. RIGHT OUTER JOIN

조인 수행시 LEFT JOIN과 반대로 우측 테이블이 기준이 되어 결과를 생성한다. 즉, TABLE A와 B가 있을 때(TABLE 'B'가 기준이 됨), A와 B를 비교해서 A의 JOIN 칼럼에서 같은 값이 있을 때 그 해당 데이터를 가져오고, A의 JOIN 칼럼에서 같은 값이 없는 경우에는 A 테이블에서 가져오는 칼럼들은 NULL 값으로 채운다. 그리고 RIGHT JOIN으로 OUTER 키워드를 생략해서 사용할 수 있다.

[예제] DEPT에 등록된 부서 중에는 사원이 없는 부서도 있다. DEPT와 EMP를 조인하되 사원이 없는 부서 정보도 같이 출력하도록 한다.

[예제] SELECT E.ENAME, D.DEPTNO, D.DNAME FROM EMP E RIGHT OUTER JOIN DEPT D ON E.DEPTNO = D.DEPTNO; OUTER는 생략 가능한 키워드이므로 아래 SQL은 같은 결과를 얻을 수 있다. SELECT E.ENAME, D.DEPTNO, D.DNAME, D.LOC FROM EMP E RIGHT JOIN DEPT D ON E.DEPTNO = D.DEPTNO;

[실행 결과] ENAME DEPTNO DNAME LOC ----- ------ ---------- -------- CLARK 10 ACCOUNTING NEW YORK KING 10 ACCOUNTING NEW YORK MILLER 10 ACCOUNTING NEW YORK JONES 20 RESEARCH DALLAS FORD 20 RESEARCH DALLAS ADAMS 20 RESEARCH DALLAS SMITH 20 RESEARCH DALLAS SCOTT 20 RESEARCH DALLAS WARD 30 SALES CHICAGO TURNER 30 SALES CHICAGO ALLEN 30 SALES CHICAGO JAMES 30 SALES CHICAGO BLAKE 30 SALES CHICAGO MARTIN 30 SALES CHICAGO 40 OPERATIONS BOSTON 15개의 행이 선택되었다.

INNER JOIN이라면 사원 정보와 함께 사원이 배정된 3개의 부서 정보와 14명의 사원 정보만 출력 되었겠지만, RIGHT OUTER JOIN을 사용하였기 때문에 사원이 배정되지 않은 부서번호 40의 OPERATIONS 부서의 LOC 정보까지 출력되었다.

##### 다. FULL OUTER JOIN

조인 수행시 좌측, 우측 테이블의 모든 데이터를 읽어 JOIN하여 결과를 생성한다. 즉, TABLE A와 B가 있을 때(TABLE 'A', 'B' 모두 기준이 됨), RIGHT OUTER JOIN과 LEFT OUTER JOIN의 결과를 합집합으로 처리한 결과와 동일하다. 단, UNION ALL이 아닌 UNION 기능과 같으므로 중복되는 데이터는 삭제한다. (UNION ALL과 UNION에 대해서는 다음 절에서 설명하도록 한다.) 그리고 FULL JOIN으로 OUTER 키워드를 생략해서 사용할 수 있다.

[예제] DEPT 테이블과 DEPT_TEMP 테이블의 FULL OUTER JOIN 사례를 만들기 위해 DEPT_TEMP의 DEPTNO를 수정한다. 결과적으로 DEPT_TEMP 테이블의 새로운 DEPTNO 데이터는 DETP 테이블의 DEPTNO와 2건은 동일하고 2건은 새로운 DEPTNO가 생성된다.

[예제] UPDATE DEPT_TEMP SET DEPTNO = DEPTNO + 20; SELECT * FROM DEPT_TEMP;

[실행 결과] DEPTNO DNAME LOC ------ ---------- ---------- 30 ACCOUNTING NEW YORK 40 R&D DALLAS 50 MARKETING CHICAGO 60 OPERATIONS BOSTON 4개의 행이 선택되었다.

[예제] DEPTNO 기준으로 DEPT와 DEPT_TEMP 데이터를 FULL OUTER JOIN으로 출력한다. 예제에 사용된 UNION(중복 데이터는 제거됨)은 다음 절에서 설명하도록 한다.

[예제] SELECT * FROM DEPT FULL OUTER JOIN DEPT_TEMP ON DEPT.DEPTNO = DEPT_TEMP.DEPTNO; OUTER는 생략 가능한 키워드이므로 아래 SQL은 같은 결과를 얻을 수 있다. SELECT * FROM DEPT FULL JOIN DEPT_TEMP ON DEPT.DEPTNO = DEPT_TEMP.DEPTNO; 위 SQL과 아래 SQL은 같은 결과를 얻을 수 있다. SELECT L.DEPTNO, L.DNAME, L.LOC, R.DEPTNO, R.DNAME, R.LOC FROM DEPT L LEFT OUTER JOIN DEPT_TEMP R ON L.DEPTNO = R.DEPTNO UNION SELECT L.DEPTNO, L.DNAME, L.LOC, R.DEPTNO, R.DNAME, R.LOC FROM DEPT L RIGHT OUTER JOIN DEPT_TEMP R ON L.DEPTNO = R.DEPTNO;

[실행 결과] DEPTNO DNAME LOC DEPTNO DNAME LOC ------ ---------- -------- ------ ----------- ------ 10 ACCOUNTING NEW YORK 20 RESEARCH DALLAS 30 SALES CHICAGO 30 ACCOUNTING NEW YORK 40 OPERATIONS BOSTON 40 R&D DALLAS 50 MARKETING CHICAGO 60 OPERATIONS BOSTON 6개의 행이 선택되었다.

INNER JOIN이라면 부서번호가 동일한 30, 40 부서의 2개 정보만 출력되었겠지만, FULL OUTER JOIN을 사용하였기 때문에 DEPT 테이블에만 있는 부서번호 10, 20의부서와 DEPT_TEMP 테이블에만 있는 부서번호 50, 60의 부서 정보까지 같이 출력되었다.

#### 9. INNER vs OUTER vs CROSS JOIN 비교

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_203.jpg)

첫 번째, INNER JOIN의 결과는 다음과 같다. 양쪽 테이블에 모두 존재하는 키 값이 B-B, C-C 인 2건이 출력된다. 두 번째, LEFT OUTER JOIN의 결과는 다음과 같다. TAB1을 기준으로 키 값 조합이 B-B, C-C, D-NULL, E-NULL 인 4건이 출력된다. 세 번째, RIGHT OUTER JOIN의 결과는 다음과 같다. TAB2를 기준으로 키 값 조합이 NULL-A, B-B, C-C 인 3건이 출력된다. 네 번째, FULL OUTER JOIN의 결과는 다음과 같다. 양쪽 테이블을 기준으로 키 값 조합이 NULL-A, B-B, C-C, D-NULL, E-NULL 인 5건이 출력된다. 다섯 번째, CROSS JOIN의 결과는 다음과 같다. JOIN 가능한 모든 경우의 수를 표시하지만 단, OUTER JOIN은 제외한다. 양쪽 테이블 TAB1과 TAB2의 데이터를 곱한 개수인 4 * 3 = 12건이 추출됨 키 값 조합이 B-A, B-B, B-C, C-A, C-B, C-C, D-A, D-B, D-C, E-A, E-B, E-C 인 12건이 출력된다.
## 집합 연산자
두 개 이상의 테이블에서 조인을 사용하지 않고 연관된 데이터를 조회하는 방법 중에 또 다른 방법이 있는데 그 방법이 바로 집합 연산자(Set Operator)를 사용하는 방법이다. 기존의 조인에서는 FROM 절에 검색하고자 하는 테이블을 나열하고, WHERE 절에 조인 조건을 기술하여 원하는 데이터를 조회할 수 있었다. 하지만 집합 연산자는 여러 개의 질의의 결과를 연결하여 하나로 결합하는 방식을 사용한다. 즉, 집합 연산자는 2개 이상의 질의 결과를 하나의 결과로 만들어 준다. 일반적으로 집합 연산자를 사용하는 상황은 서로 다른 테이블에서 유사한 형태의 결과를 반환하는 것을 하나의 결과로 합치고자 할 때와 동일 테이블에서 서로 다른 질의를 수행하여 결과를 합치고자 할 때 사용할 수 있다. 이외에도 튜닝관점에서 실행계획을 분리하고자 하는 목적으로도 사용할 수 있다. 집합 연산자를 사용하기 위해서는 다음 제약조건을 만족해야 한다. SELECT 절의 칼럼 수가 동일하고 SELECT 절의 동일 위치에 존재하는 칼럼의 데이터 타입이 상호 호환 가능(반드시 동일한 데이터 타입일 필요는 없음)해야 한다. 그렇지 않으면 데이터베이스가 오류를 반환한다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_204.jpg)

집합 연산자는 개별 SQL문의 결과 집합에 대해 합집합(UNION/UNION ALL), 교집합(INTERSECT), 차집합(EXCEPT)으로 집합간의 관계를 가지고 작업을 한다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_205.jpg)

집합 연산자를 가지고 연산한 결과는 [그림 Ⅱ-2-5]와 같다. [그림 Ⅱ-2-5]의 왼쪽에 존재하는 R1, R2는 각각의 SQL문을 실행해서 생성된 개별 결과 집합을 의미한다. [그림 Ⅱ-2-5]에서 보면 알 수 있듯이 UNION ALL을 제외한 다른 집합 연산자에서는 SQL문의 결과 집합에서 먼저 중복된 건을 배제하는 작업을 수행한 후에 집합 연산을 적용한다(논리적인 관점의 처리임). UNION 연산에서 R1 = {1, 2, 3, 5}, R2 = {1, 2, 3, 4}가 되고, 이것의 합집합(R1 ∪ R2)의 결과는 {1, 2, 3, 4, 5}이다. UNION ALL 연산은 중복에 대한 배제 없이 2개의 결과 집합을 단순히 합친 것과 동일한 결과이다. UNION ALL의 결과는 {1, 1, 1, 2, 2, 3, 3, 5, 1, 1, 2, 2, 2, 3, 4}이다. INTERSECT 연산에서 R1 = {1, 2, 3, 5}, R2 = {1, 2, 3, 4}가 되어, 이것의 교집합(R1 ∩ R2)의 결과는 {1, 2, 3}이다. EXCEPT 연산에서는 R1 = {1, 2, 3, 5}, R2 = {1, 2, 3, 4}가 되고, 이것의 차집합(R1 R2)의 결과는 {5}이다. EXCEPT 연산에서는 순서가 중요하다. 만약 순서가 바뀌어서 R2 R1의 차집합이었다면 결과는 {4}가 된다. 집합 연산자를 사용하여 만들어지는 SQL문의 형태는 다음과 같다.

SELECT 칼럼명1, 칼럼명2, ... FROM 테이블명1 [WHERE 조건식 ] [[GROUP BY 칼럼(Column)이나 표현식 [HAVING 그룹조건식 ] ] 집합 연산자 SELECT 칼럼명1, 칼럼명2, ... FROM 테이블명2 [WHERE 조건식 ] [[GROUP BY 칼럼(Column)이나 표현식 [HAVING 그룹조건식 ] ] [ORDER BY 1, 2 [ASC또는 DESC ] ; SELECT PLAYER_NAME 선수명, BACK_NO 백넘버 FROM PLAYER WHERE TEAM_ID = 'K02' UNION SELECT PLAYER_NAME 선수명, BACK_NO 백넘버 FROM PLAYER WHERE TEAM_ID = 'K07' ORDER BY 1;

집합 연산자는 사용상의 제약조건을 만족한다면 어떤 형태의 SELECT문이라도 이용할 수 있다. 집합 연산자는 여러 개의 SELECT문을 연결하는 것에 지나지 않는다. ORDER BY는 집합 연산을 적용한 최종 결과에 대한 정렬 처리이므로 가장 마지막 줄에 한번만 기술한다. 아래 질문에 대해 집합 연산자를 사용하여 처리하는 방법을 알아보자.

[집합 연산자를 연습하기 위한 질문]

1) K-리그 소속 선수들 중에서 소속이 삼성블루윙즈팀인 선수들과전남드레곤즈팀인 선수들에 대한 내용을 모두보고 싶다. 2) K-리그 소속 선수들 중에서 소속이 삼성블루윙즈팀인 선수들과 포지션이 골키퍼(GK)인 선수들을 모두 보고 싶다. 3) K-리그 소속 선수들에 대한 정보 중에서 포지션별 평균키와 팀별 평균키를 알고 싶다. 4) K-리그 소속 선수를 중에서 소속이 삼성블루윙즈팀이면서 포지션이 미드필더(MF)가 아닌 선수들의 정보를 보고 싶다. 5) K-리그 소속 선수들 중에서 소속이 삼성블루윙즈팀이면서 포지션이 골키퍼(GK)인 선수들의 정보를 보고 싶다.

SQL문을 작성하기에 전에 [집합 연산자를 연습하기 위한 질문]을 다음과 같이 집합 연산자를 사용 형태로 해석할 수 있다.

[질문을 집합 연산의 개념으로 해석한 결과]

1) K-리그 소속 선수 중 소속이 삼성블루윙즈팀인 선수들의 집합과K-리그 소속 선수 중 소속이 전남드레곤즈팀인 선수들의 집합의 합집합 2) K-리그 소속 선수 중 소속이 삼성블루윙즈팀인 선수들의 집합과K-리그 소속 선수 중 포지션이 골키퍼(GK)인 선수들의 집합의 합집합 3) K-리그 소속 선수 중 포지션별 평균키에 대한 집합과K-리그 소속 선수 중 팀별 평균키에 대한 집합의 합집합 4) K-리그 소속 선수 중 소속이 삼성블루윙즈팀인 선수들의 집합과K-리그 소속 선수 중 포지션이 미드필더(MF))인 선수들의 집합의 차집합 5) K-리그 소속 선수 중 소속이 삼성블루윙즈팀인 선수들의 집합과K-리그 소속 선수 중 포지션이 골키퍼(GK)인 선수들의 집합의 교집합

위의 결과를 집합 연산자를 사용하여 SQL문을 작성하고 그 결과를 확인해 보도록 하자. 먼저 첫 번째 질문에 대한 SQL문을 작성하고 실행해 보자.

[질문1] 1) K-리그 소속 선수들 중에서 소속이 삼성블루윙즈팀인 선수들과전남드레곤즈팀인 선수들에 대한 내용을 모두보고 싶다. 1) K-리그 소속 선수 중 소속이 삼성블루윙즈팀인 선수들의 집합과K-리그 소속 선수 중 소속이 전남드레곤즈팀인 선수들의 집합의 합집합

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' UNION SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K07'

[실행 결과] 팀코드 선수명 포지션 백넘버 키 ---- ---- ---- ---- -- K02 가비 MF 10 177 K02 강대희 MF 26 174 K02 고종수 MF 22 176 K02 고창현 MF 8 170 K02 김강진 DF 43 181 K07 강철 DF 3 178 K07 김반 MF 14 174 K07 김영수 MF 30 175 K07 김정래 GK 33 185 K07 김창원 DF 5 183 100개의 행이 선택되었다.

첫 번째 질문에 대한 SQL문에서 삼성 블루윙즈팀인 선수들과 전남 드레곤즈팀의 선수들의 합집합이라는 것은 WHERE 절에 IN 또는 OR 연산자로도 변환이 가능하다. 다만 IN 또는 OR 연산자를 사용할 경우에는 결과의 표시 순서가 달라질 수 있다. 집합이라는 관점에서는 결과가 표시되는 순서가 틀렸다고 두 집합이 서로 다르다고 말할 수 없다. 만약, 결과의 동일한 표시 순서를 원한다면 ORDER BY절을 사용해서 명시적으로 정렬 순서를 정의하는 것이 바람직하다. 첫 번째 질문에 대한 SQL문을 IN 또는 OR 연산자를 사용한 SQL문으로 변경해 보고 결과의 표시 순서가 다름을 확인해 보자.

[예제] (비교) SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' OR TEAM_ID = 'K07'; SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID IN ('K02', 'K07');

[실행 결과] 팀 선수명 포지션 백넘버 키 ---- ----- ---- ----- --- K07 김회택 TM K07 서현옥 TC K07 정상호 TC K07 최철우 TC K07 정영광 GK 41 185 K02 정호 TM K02 왕선재 TC K02 코샤 TC K02 윤성효 TC K02 정광수 GK 41 182 100개의 행이 선택되다.

두 번째 질문에 대해 SQL문을 작성하고 결과를 확인해 보자.

[질문2] 2) K-리그 소속 선수들 중에서 소속이 삼성블루윙즈팀인 선수들과 포지션이 골키퍼(GK)인 선수들을 모두 보고 싶다. 2) K-리그 소속 선수 중 소속이 삼성블루윙즈팀인 선수들의 집합과K-리그 소속 선수 중 포지션이 골키퍼(GK)인 선수들의 집합의 합집합

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' UNION SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE POSITION = 'GK';

[실행 결과] 팀코드 선수명 포지션 백넘버 키 ---- ----- ----- ---- -- K01 권정혁 GK 1 195 K01 서동명 GK 21 196 K01 양지원 GK 45 181 K01 이무림 GK 31 185 K01 최창주 GK 40 187 K02 가비 MF 10 177 K02 강대희 MF 26 174 K02 고종수 MF 22 176 K02 고창현 MF 8 170 K02 김강진 DF 43 181 88개의 행이 선택되었다.

두 번째 질문에 대한 실행 결과는 첫 번째 실행 결과와 비교해 보면 집합의 대상만 차이가 날뿐 다른 점은 없다. 마찬가지로 두 번째 질문에 대해 OR 연산자를 사용한 SQL문으로 변경하면 다음과 같다. 여기서는 서로 다른 칼럼에 조건을 사용했기 때문에 IN 연산자를 사용할 수 없다.

[예제] (비교) SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' OR POSITION = 'GK';

만약, 두 번째 질문에 대한 SQL문에서 UNION이라는 집합 연산자 대신에 UNION ALL이라는 집합 연산자를 사용하면 어떻게 될지 한번 수행해 보자.

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' UNION ALL SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE POSITION = 'GK';

[실행 결과] 팀코드 선수명 포지션 백넘버 키 ---- ----- ---- ---- --- K02 정호 TM K02 왕선재 TC K02 코샤 TC K02 윤성효 TC K02 정광수 GK 41 182 K04 남현우 GK 31 180 K04 김충호 GK 60 185 K04 이현 GK 1 192 K04 한동진 GK 21 183 K10 강성일 GK 30 182 92개의 행이 선택되었다.

수행 결과에서 알 수 있듯이 결과 건수가 UNION은 88건이었으나 UNION ALL은 92건으로 결과 건수가 늘어났다. 두 SQL문의 결과가 서로 다르다. 결과가 다른 이유는 UNION은 결과에서 중복이 존재할 경우 중복을 제외시키지만 UNION ALL은 각각의 질의 결과를 단순히 결합시켜 줄 뿐 중복된 결과를 제외시키지 않기 때문이다. 이와 같이 결과 집합에 중복이 존재하면 UNION과 UNION ALL의 결과는 달라진다. UNION ALL에서 중복된 결과들을 확인해 보고자 할 때는 ORDER BY절을 사용하면 용이하다. 아래 SQL문을 통해 중복된 결과를 확인해 보자.

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' UNION ALL SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE POSITION = 'GK' ORDER BY 1, 2, 3, 4, 5;

[실행 결과] 팀코드 선수명 포지션 백넘버 키 ---- ---- ---- ---- --- K02 김운재 GK 1 182 K02 김운재 GK 1 182 K02 정광수 GK 41 182 K02 정광수 GK 41 182 K02 조범철 GK 21 185 K02 조범철 GK 21 185 K02 최호진 GK 31 190 K02 최호진 GK 31 190 92개의 행이 선택되었다.

결과에서 삼성블루윙즈팀(K02)에서 포지션이 골키퍼(GK)인 사람이 중복 표시되째 질문에 대한 SQL문을 작성하고 결과를 확인해 보자.

[질문3] 3) K-리그 소속 선수들에 대한 정보 중에서 포지션별 평균키와 팀별 평균키를 알고 싶다. 3) K-리그 소속 선수 중 포지션별 평균키에 대한 집합과K-리그 소속 선수 중 팀별 평균키에 대한 집합의 합집합

[예제] SELECT 'P' 구분코드, POSITION 포지션, AVG(HEIGHT) 평균키 FROM PLAYER GROUP BY POSITION UNION SELECT 'T' 구분코드, TEAM_ID 팀명, AVG(HEIGHT) 평균키 FROM PLAYER GROUP BY TEAM_ID ORDER BY 1;

[실행 결과] 구분코드 포지션 평균키 ------ ----- -------- P DF 180.409 P FW 179.91 P GK 186.256 P MF 176.309 P TC 178.833 T K01 180.089 T K02 179.067 T K03 179.911 T K04 180.511 T K05 180.422 23개의 행이 선택되었다.

세 번째 질문에서는 평균키에 대한 값들의 합집합을 구하는 것이다. 합집합을 구하기 위해 SQL문에서 그룹함수를 사용했다. 그룹함수도 집합 연산자에서 사용이 가능하다는 것을 알 수 있다. 또한 실제로 테이블에는 존재하지 않지만 결과 행을 구분하기 위해 SELECT 절에 칼럼('구분코드')을 추가할 수 있다는 것을 알 수 있다. 이와 같이 목적을 위해 SELECT 절에 임의의 칼럼을 추가하는 것은 다른 모든 SQL문에서 적용 가능하다. 집합 연산자의 결과를 표시할 때 HEADING 부분은 첫 번째 SQL문에서 사용된 HEADING이 적용된다는 것을 알 수 있다. SQL문에서 첫 번째 SELECT 절에서는 '포지션' HEADING을 사용하였고 두 번째 SELECT 절에서는 '팀명' HEADING을 사용하였다. 그러나 결과에는 '포지션' HEADING으로 표시되었다.

네 번째 질문인 삼성 블루윙즈팀인 집합과 포지션이 미드필더(MF)인 선수들의 차집합에 대한 SQL문을 작성하고 결과를 확인해 보자.

[질문4] 4) K-리그 소속 선수를 중에서 소속이 삼성블루윙즈팀이면서 포지션이 미드필더(MF)가 선수들의 정보를 보고 싶다. 4) K-리그 소속 선수 중 소속이 삼성블루윙즈팀인 선수들의 집합과K-리그 소속 선수 중 포지션이 미드필더(MF))인 선수들의 집합의 차집합

[예제] Oracle SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' MINUS SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE POSITION = 'MF' ORDER BY 1, 2, 3, 4, 5;

SQL Server에서는 MINUS대신 EXCEPT를 사용할 수 있다.

[실행 결과] 팀코드 선수명 포지션 백넘버 키 ---- ------- ----- ---- -- K02 김강진 DF 43 181 K02 김관희 FW 39 180 K02 김만근 FW 34 177 K02 김병국 DF 2 183 K02 김병근 DF 3 175 K02 왕선재 TC K02 윤성효 TC K02 윤화평 FW 42 182 K02 이성용 DF 20 173 K02 정광수 GK 41 182 31개의 행이 선택되었다.

차집합은 앞의 집합의 결과에서 뒤의 집합의 결과를 빼는 것이다. 이번 SQL문은 삼성블루윙즈팀의 선수들 중에서 포지션이 미드필더(MF)인 선수들의 정보를 빼는 것이다. 해당 SQL문은 다른 형태의 SQL문으로 변경 가능하다. EXCEPT 연산자의 앞에 오는 SQL문의 조건은 만족하고 뒤에 오는 SQL문의 조건은 만족하지 않는 SQL문과 동일한 결과를 얻을 수 있다. 그러므로 EXCEPT 연산자를 사용하지 않고 논리 연산자를 이용하여 동일한 결과의 SQL문을 작성할 수 있다. SQL문을 실행하여 결과가 동일한지 직접 확인해 보자.

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' AND POSITION <> 'MF' ORDER BY 1, 2, 3, 4, 5;

MINUS 연산자는 NOT EXISTS 또는 NOT IN 서브쿼리를 이용한 SQL문으로도 변경 가능하다. (NOT EXISTS와 NOT IN에 대한 설명은 제5절 서브쿼리에서 참고)

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER X WHERE X.TEAM_ID = 'K02' AND NOT EXISTS (SELECT 1 FROM PLAYER Y WHERE Y.PLAYER_ID = X.PLAYER_ID AND POSITION = 'MF') ORDER BY 1, 2, 3, 4, 5; SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' AND PLAYER_ID NOT IN (SELECT PLAYER_ID FROM PLAYER WHERE POSITION = 'MF') ORDER BY 1, 2, 3, 4, 5;

이제 마지막으로 삼성블루윙즈팀이면서 포지션이 골키퍼인 선수들인 교집합을 얻기 위한 SQL문을 작성해 보자.

[질문 5] 5) K-리그 소속 선수들 중에서 소속이 삼성블루윙즈팀이면서 포지션이 골키퍼(GK)인 선수들의 정보를 보고 싶다. 5) K-리그 소속 선수 중 소속이 삼성블루윙즈팀인 선수들의 집합과K-리그 소속 선수 중 포지션이 골키퍼(GK)인 선수들의 집합의 교집합

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' INTERSECT SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE POSITION = 'GK' ORDER BY 1, 2, 3, 4, 5;

[실행 결과] 팀코드 선수명 포지션 백넘버 키 ---- ------ ----- ---- -- K02 김운재 GK 1 182 K02 정광수 GK 41 182 K02 조범철 GK 21 185 K02 최호진 GK 31 190 4개의 행이 선택되었다.

교집합의 결과는 소속이 삼성 블루윙즈팀인 선수의 집합이면서 포지션이 골키퍼인 집합인 두 개의 조건을 만족하는 집합이다. 이것은 INTERSECT 연산자의 앞에 오는 SQL문의 조건은 만족하면서 뒤의 SQL문의 조건을 만족하는 것과 동일한 결과를 얻을 수 있다. 다음과 같이 INTERSECT 연산자를 사용하지 않고도 논리 연산자만으로 결과가 동일한 SQL문을 작성할 수 있다. SQL문을 실행하여 결과가 동일한지 직접 확인해 보자.

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' AND POSITION = 'GK' ORDER BY 1, 2, 3, 4, 5;

INTERSECT 연산자는 EXISTS 또는 IN 서브쿼리를 이용한 SQL문으로 변경 가능하다. (EXISTS와 IN에 대한 설명은 제5절 서브쿼리에서 참고)

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER X WHERE X.TEAM_ID = 'K02' AND EXISTS (SELECT 1 FROM PLAYER Y WHERE Y.PLAYER_ID = X.PLAYER_ID AND Y.POSITION = 'GK') ORDER BY 1, 2, 3, 4, 5;

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE TEAM_ID = 'K02' AND PLAYER_ID IN (SELECT PLAYER_ID FROM PLAYER WHERE POSITION = 'GK') ORDER BY 1, 2, 3, 4, 5;
## 계층형 질의와 셀프 조인
#### 1. 계층형 질의

테이블에 계층형 데이터가 존재하는 경우 데이터를 조회하기 위해서 계층형 질의(Hierarchical Query)를 사용한다. 계층형 데이터란 동일 테이블에 계층적으로 상위와 하위 데이터가 포함된 데이터를 말한다. 예를 들어, 사원 테이블에서는 사원들 사이에 상위 사원(관리자)과 하위 사원 관계가 존재하고 조직 테이블에서는 조직들 사이에 상위 조직과 하위 조직 관계가 존재한다. 엔터티를 순환관계 데이터 모델로 설계할 경우 계층형 데이터가 발생한다. 순환관계 데이터 모델의 예로는 조직, 사원, 메뉴 등이 있다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_206.jpg)

[그림 Ⅱ-2-6]은 사원에 대한 순환관계 데이터 모델을 표현한 것이다. (2)계층형 구조에서 A의 하위 사원은 B, C이고 B 밑에는 하위 사원이 없고 C의 하위 사원은 D, E가 있다. 계층형 구조를 데이터로 표현한 것이 (3)샘플 데이터이다. 계층형 데이터 조회는 DBMS 벤더와 버전에 따라 다른 방법으로 지원한다. 여기서는 Oracle과 SQL Server 기준으로 설명한다.

##### 가. Oracle 계층형 질의

Oracle은 계층형 질의를 지원하기 위해서 [그림 Ⅱ-2-7]과 같은 계층형 질의 구문을 제공한다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_207.jpg)

- START WITH절은 계층 구조 전개의 시작 위치를 지정하는 구문이다. 즉, 루트 데이터를 지정한다.(액세스) - CONNECT BY절은 다음에 전개될 자식 데이터를 지정하는 구문이다. 자식 데이터는 CONNECT BY절에 주어진 조건을 만족해야 한다.(조인) - PRIOR : CONNECT BY절에 사용되며, 현재 읽은 칼럼을 지정한다. PRIOR 자식 = 부모 형태를 사용하면 계층구조에서 자식 데이터에서 부모 데이터(자식 → 부모) 방향으로 전개하는 순방향 전개를 한다. 그리고 PRIOR 부모 = 자식 형태를 사용하면 반대로 부모 데이터에서 자식 데이터(부모 → 자식) 방향으로 전개하는 역방향 전개를 한다. - NOCYCLE : 데이터를 전개하면서 이미 나타났던 동일한 데이터가 전개 중에 다시 나타난다면 이것을 가리켜 사이클(Cycle)이 형성되었다라고 말한다. 사이클이 발생한 데이터는 런타임 오류가 발생한다. 그렇지만 NOCYCLE를 추가하면 사이클이 발생한 이후의 데이터는 전개하지 않는다. - ORDER SIBLINGS BY : 형제 노드(동일 LEVEL) 사이에서 정렬을 수행한다. - WHERE : 모든 전개를 수행한 후에 지정된 조건을 만족하는 데이터만 추출한다.(필터링)

Oracle은 계층형 질의를 사용할 때 다음과 같은 가상 칼럼(Pseudo Column)을 제공한다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_208.jpg)

다음은 [그림 Ⅱ-2-6]의 (3)샘플 데이터를 계층형 질의 구문을 이용해서 조회한 것이다. 여기서는 결과 데이터를 들여쓰기 하기 위해서 LPAD 함수를 사용하였다.

[예제] SELECT LEVEL, LPAD(' ', 4 * (LEVEL-1)) || 사원 사원, 관리자, CONNECT_BY_ISLEAF ISLEAF FROM 사원 START WITH 관리자 IS NULL CONNECT BY PRIOR 사원 = 관리자;

[실행 결과] LEVEL 사원 관리자 ISLEAF ----- -------- ----- ------ 1 A 0 2 B A 1 2 C A 0 3 D C 1 3 E C 1

A는 루트 데이터이기 때문에 레벨이 1이다. A의 하위 데이터인 B, C는 레벨이 2이다. 그리고 C의 하위 데이터인 D, E는 레벨이 3이다. 리프 데이터는 B, D, E이다. 관리자 → 사원 방향을 전개이기 때문에 순방향 전개이다. [그림 Ⅱ-2-8]은 계층형 질의에 대한 논리적인 실행 모습이다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_209.jpg)

다음 예제는 사원 'D'로부터 자신의 상위관리자를 찾는 역방향 전개의 예이다.

[예제] SELECT LEVEL, LPAD(' ', 4 * (LEVEL-1)) || 사원 사원, 관리자, CONNECT_BY_ISLEAF ISLEAF FROM 사원 START WITH 사원 = 'D' CONNECT BY PRIOR 관리자 = 사원;

[실행 결과] LEVEL 사원 관리자 ISLEAF ----- --------- ----- ----- 1 D C 0 2 C A 0 3 A 1

본 예제는 역방향 전개이기 때문에 하위 데이터에서 상위 데이터로 전개된다. 결과를 보면 내용을 제외하고 표시 형태는 순방향 전개와 동일하다. D는 루트 데이터이기 때문에 레벨이 1이다. D의 상위 데이터인 C는 레벨이 2이다. 그리고 C의 상위 데이터인 A는 레벨이 3이다. 리프 데이터는 A이다. 루트 및 레벨은 전개되는 방향에 따라 반대가 됨을 알 수 있다. [그림 Ⅱ-2-9]는 역방향 전개에 대한 계층형 질의에 대한 논리적인 실행 모습이다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_210.jpg)

Orcle은 계층형 질의를 사용할 때 사용자 편의성을 제공하기 위해서 [표 Ⅱ-2-3]과 같은 함수를 제공한다

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_211.jpg)

SYS_CONNECT_BY_PATH, CONNECT_BY_ROOT를 사용한 예는 다음과 같다.

[예제] SELECT CONNECT_BY_ROOT 사원 루트사원, SYS_CONNECT_BY_PATH(사원, '/') 경로, 사원, 관리자 FROM 사원 START WITH 관리자 IS NULL CONNECT BY PRIOR 사원 = 관리자

[실행 결과] 루트사원 경로 사원 관리자 ------- ------- ---- ----- A /A A A /A/B B A A /A/C C A A /A/C/D D C A /A/C/E E C

START WITH를 통해 추출된 루트 데이터가 1건 이기 때문에 루트사원은 모두 A이다. 경로는 루트로부터 현재 데이터까지의 경로를 표시한다. 예를 들어, D의 경로는 A → C → D 이다.

##### 나. SQL Server 계층형 질의

SQL Server 2000 버전까지는 계층형 질의를 작성 계층적 구조를 가진 데이터는 저장 프로시저를 재귀 호출하거나 While 루프 문에서 임시 테이블을 사용하는 등 (순수한 쿼리가 아닌) 프로그램 방식으로 전개해야만 했다. 그러나 SQL Server 2005 버전부터는 하나의 질의로 원하는 결과를 얻을 수 있게 되었다. 먼저, Northwind 데이터베이스에 접속하여 Employees 테이블의 데이터를 조회해 보자.

USE NORTHWIND GO SELECT EMPLOYEEID, LASTNAME, FIRSTNAME, REPORTSTO FROM EMPLOYEES GO ************************************************************************** EmployeeID LastName FirstName ReportsTo --------- -------- ------- -------- 1 Davolio Nancy 2 2 Fulle Andrew NULL 3 Leverling Janet 2 4 Peacock Margaret 2 5 Buchanan Steven 2 6 Suyama Michael 5 7 King Robert 5 8 Callahan Laura 2 9 Dodsworth Anne 5 (9개 행 적용됨)

총 9개 로우가 있는데, ReportsTo 칼럼이 상위 사원에 해당하며 EmployeeID 칼럼과 재귀적 관계를 맺고 있다. EmployeeID가 2인 Fuller 사원을 살펴보면, ReportsTo 칼럼 값이 NULL이므로 계층 구조의 최상위에 있음을 알 수 있다. CTE(Common Table Expression)를 재귀 호출함으로써 Employees 데이터의 최상위부터 시작해 하위 방향으로 계층 구조를 전개하도록 작성한 쿼리와 결과는 다음과 같다.

WITH EMPLOYEES_ANCHOR AS ( SELECT EMPLOYEEID, LASTNAME, FIRSTNAME, REPORTSTO, 0 AS LEVEL FROM EMPLOYEES WHERE REPORTSTO IS NULL /* 재귀 호출의 시작점 */ UNION ALL SELECT R.EMPLOYEEID, R.LASTNAME, R.FIRSTNAME, R.REPORTSTO, A.LEVEL + 1 FROM EMPLOYEES_ANCHOR A, EMPLOYEES R WHERE A.EMPLOYEEID = R.REPORTSTO ) SELECT LEVEL, EMPLOYEEID, LASTNAME, FIRSTNAME, REPORTSTO FROM EMPLOYEES_ANCHOR GO ************************************************************************** Level EmployeeID LastName FirstName ReportsTo ---- -------- ------- ----- -------- 0 2 Fuller Andrew NULL 1 1 Davolio Nancy 2 1 3 Leverling Janet 2 1 4 Peacock Margaret 2 1 5 Buchanan Steven 2 1 8 Callahan Laura 2 2 6 Suyama Michael 5 2 7 King Robert 5 2 9 Dodsworth Anne 5 (9개 행 적용됨)

WITH 절의 CTE 쿼리를 보면, UNION ALL 연산자로 쿼리 두 개를 결합했다. 둘 중 위에 있는 쿼리를 ‘앵커 멤버’(Anchor Member)라고 하고, 아래에 있는 쿼리를 ‘재귀 멤버’(Recursive Member)라고 한다. 아래는 재귀적 쿼리의 처리 과정이다.

1. CTE 식을 앵커 멤버와 재귀 멤버로 분할한다. 2. 앵커 멤버를 실행하여 첫 번째 호출 또는 기본 결과 집합(T0)을 만든다. 3. Ti는 입력으로 사용하고 Ti+1은 출력으로 사용하여 재귀 멤버를 실행한다. 4. 빈 집합이 반환될 때까지 3단계를 반복한다. 5. 결과 집합을 반환한다. 이것은 T0에서 Tn까지의 UNION ALL이다.

정리하자면 다음과 같다. 먼저, 앵커 멤버가 시작점이자 Outer 집합이 되어 Inner 집합인 재귀 멤버와 조인을 시작한다. 이어서, 앞서 조인한 결과가 다시 Outer 집합이 되어 재귀 멤버와 조인을 반복하다가 조인 결과가 비어 있으면 즉, 더 조인할 수 없으면 지금까지 만들어진 결과 집합을 모두 합하여 리턴한다. [그림 Ⅱ-2-10]에 있는 조직도를 쿼리로 출력했을 때, 대부분 사용자는 아래와 같은 결과를 기대할 것이다.(보기 편하도록 각 로우 앞쪽에 자신의 레벨만큼 빈칸을 삽입했다.)

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_212.jpg)

EmployeeID ManagerID -------- --------- 1000 NULL 1100 1000 1110 1100 1120 1100 1121 1120 1122 1120 1200 1000 1210 1200 1211 1210 1212 1210 1220 1200 1221 1220 1222 1220 1300 1000

아래에 t_emp 데이터의 최상위부터 시작해 하위 방향으로 계층 구조를 전개하도록 작성한 쿼리와 그 결과이다.

WITH T_EMP_ANCHOR AS ( SELECT EMPLOYEEID, MANAGERID, 0 AS LEVEL FROM T_EMP WHERE MANAGERID IS NULL /* 재귀 호출의 시작점 */ UNION ALL SELECT R.EMPLOYEEID, R.MANAGERID, A.LEVEL + 1 FROM T_EMP_ANCHOR A, T_EMP R WHERE A.EMPLOYEEID = R.MANAGERID ) SELECT LEVEL, REPLICATE(' ', LEVEL) + EMPLOYEEID AS EMPLOYEEID, MANAGERID FROM T_EMP_ANCHOR GO ************************************************************************** Level EmployeeID ManagerID --- ------- --------- 0 1000 NULL 1 1100 1000 1 1200 1000 1 1300 1000 2 1210 1200 2 1220 1200 3 1221 1220 3 1222 1220 3 1211 1210

3 1212 1210 2 1110 1100 2 1120 1100 3 1121 1120 3 1122 1120 (14개 행 적용됨

보다시피, 계층 구조를 단순히 하위 방향으로 전개했을 뿐 [그림 Ⅱ-2-10]에 있는 조직도와는 많이 다른 모습이다. 앞서 보았듯이, CTE 재귀 호출로 만들어낸 계층 구조는 실제와 다른 모습으로 출력된다. 따라서 조직도와 같은 모습으로 출력하려면 order by 절을 추가해 원하는 순서대로 결과를 정렬해야 한다. 실제 조직도와 같은 모습의 결과를 출력하도록, CTE에 Sort라는 정렬용 칼럼을 추가하고 쿼리 마지막에 order by 조건을 추가해보자.(단, 앵커 멤버와 재귀 멤버 양쪽에서 convert 함수 등으로 데이터 형식을 일치시켜야 한다.)

WITH T_EMP_ANCHOR AS ( SELECT EMPLOYEEID, MANAGERID, 0 AS LEVEL, CONVERT(VARCHAR(1000), EMPLOYEEID) AS SORT FROM T_EMP WHERE MANAGERID IS NULL /* 재귀 호출의 시작점 */ UNION ALL SELECT R.EMPLOYEEID, R.MANAGERID, A.LEVEL + 1, CONVERT(VARCHAR(1000), A.SORT + '/' + R.EMPLOYEEID) AS SORT FROM T_EMP_ANCHOR A, T_EMP R WHERE A.EMPLOYEEID = R.MANAGERID ) SELECT LEVEL, REPLICATE(' ', LEVEL) + EMPLOYEEID AS EMPLOYEEID, MANAGERID, SORT FROM T_EMP_ANCHOR ORDER BY SORT GO

CTE 안에서 Sort 칼럼에 사번(=EmployeeID)을 재귀적으로 더해 나가면 정렬 기준으로 삼을 수 있는 값이 만들어진다. 아래는 Sort 칼럼으로 정렬하여 출력한 결과인데, [그림 Ⅱ-2-10]에 있는 조직도의 모습과 일치한다.

Level EmployeeID ManagerID Sort ---- -------- -------- ------------- 0 1000 NULL 1000 1 1100 1000 1000/1100 2 1110 1100 1000/1100/1110 2 1120 1100 1000/1100/1120 3 1121 1120 1000/1100/1120/1121 3 1122 1120 1000/1100/1120/1122 1 1200 1000 1000/1200 2 1210 1200 1000/1200/1210 3 1211 1210 1000/1200/1210/1211 3 1212 1210 1000/1200/1210/1212 2 1220 1200 1000/1200/1220 3 1221 1220 1000/1200/1220/1221 3 1222 1220 1000/1200/1220/1222 1 1300 1000 1000/1300 (14개 행 적용됨)

가상의 Sort 칼럼을 추가해 정렬하는 게 아쉽기는 하지만, SQL Server에서 계층 구조를 실제 모습대로 출력하려면 현재(2005, 2008 버전 기준)로서는 감수해야 할 수밖에 없다.

#### 2. 셀프 조인

셀프 조인(Self Join)이란 동일 테이블 사이의 조인을 말한다. 따라서 FROM 절에 동일 테이블이 두 번 이상 나타난다. 동일 테이블 사이의 조인을 수행하면 테이블과 칼럼 이름이 모두 동일하기 때문에 식별을 위해 반드시 테이블 별칭(Alias)를 사용해야 한다. 그리고 칼럼에도 모두 테이블 별칭을 사용해서 어느 테이블의 칼럼인지 식별해줘야 한다. 이외 사항은 조인과 동일하다.

셀프 조인에 대한 기본적인 사용법은 다음과 같다

SELECT ALIAS명1.칼럼명, ALIAS명2.칼럼명, ... FROM 테이블1 ALIAS명1, 테이블2 ALIAS명2 WHERE ALIAS명1.칼럼명2 = ALIAS명2.칼럼명1;

SELECT WORKER.ID 사원번호, WORKER.NAME 사원명, MANAGER.NAME 관리자명 FROM EMP WORKER, EMP MANAGER WHERE WORKER.MGR = MANAGER.ID;

계층형 질의에서 살펴보았던 사원이라는 테이블 속에는 사원과 관리자가 모두 하나의 사원이라는 개념으로 동일시하여 같이 입력되어 있다. 이것을 이용해서 다음 문제를 셀프 조인으로 해결해 보면 다음과 같다. “자신과 상위, 차상위 관리자를 같은 줄에 표시하라.” 이 문제를 해결하기 위해서는 FROM 절에 사원 테이블을 두 번 사용해야 한다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_213.jpg)

셀프 조인은 동일한 테이블(사원)이지만 [그림 Ⅱ-2-11]과 같이 개념적으로는 두 개의 서로 다른 테이블(사원, 관리자)을 사용하는 것과 동일하다. 동일 테이블을 다른 테이블인 것처럼 처리하기 위해 테이블 별칭을 사용한다. 여기서는 E1(사원), E2(관리자) 테이블 별칭을 사용하였다. 차상위 관리자를 구하기 위해서 E1.관리자 = E2.사원 조인 조건을 사용한다. 셀프 조인을 이용한 SQL문은 다음과 같다.

[예제] SELECT E1.사원, E1.관리자, E2.관리자 차상위_관리자 FROM 사원 E1, 사원 E2 WHERE E1.관리자 = E2.사원 ORDER BY E1.사원;

[실행 결과] 사원 관리자 차상위_관리자 ---- ------ ---------- B A C A D C A E C A

자신과 자신의 직속 관리자는 동일한 행에서 데이터를 구할 수 있으나 차상위 관리자는 바로 구할 수 없다. 차상위 관리자를 구하기 위해서는 자신의 직속 관리자를 기준으로 사원 테이블과 한번 더 조인(셀프 조인)을 수행해야 한다. 결과 표시를 위해 SELECT절에 2개의 ‘관리자’ 칼럼을 사용되었다. 한 명은 자신의 직속 관리자(E1.관리자)이고 다른 한 명은 자신의 차상위 관리자(E2.관리자)이다. 결과를 보면, B와 C의 관리자는 A이고 차상위 관리자는 없다. D와 E의 관리자는 C이고 차상위 관리자는 A이다. 결과에서 A에 대한 정보는 누락되었다. 내부 조인(Inner Join)을 사용할 경우 자신의 관리자가 존재하지 않는 경우에는 관리자(E2) 테이블에서 조인할 대상이 존재하지 않기 때문에 해당 데이터는 결과에서 누락된다. 이를 방지하기 위해서는 아우터 조인을 사용해야 한다. 다음은 아우터 조인을 사용한 예이다.

[예제] SELECT E1.사원, E1.관리자, E2.관리자 차상위_관리자 FROM 사원 E1 LEFT OUTER JOIN 사원 E2 ON (E1.관리자 = E2.사원) ORDER BY E1.사원;

[실행 결과] 사원 관리자 차상위_관리자 ---- ----- ---------- A B A C A D C A E C A

아우터 조인을 사용해서 관리자가 존재하지 않는 데이터까지 모두 결과에 표시되었다.
## 서브쿼리
서브쿼리(Subquery)란 하나의 SQL문안에 포함되어 있는 또 다른 SQL문을 말한다. 서브쿼리는 알려지지 않은 기준을 이용한 검색을 위해 사용한다. 서브쿼리는 [그림 Ⅱ-2-12]와 같이 메인쿼리가 서브쿼리를 포함하는 종속적인 관계이다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_214.jpg)

조인은 조인에 참여하는 모든 테이블이 대등한 관계에 있기 때문에 조인에 참여하는 모든 테이블의 칼럼을 어느 위치에서라도 자유롭게 사용할 수 있다. 그러나 서브쿼리는 메인쿼리의 칼럼을 모두 사용할 수 있지만 메인쿼리는 서브쿼리의 칼럼을 사용할 수 없다. 질의 결과에 서브쿼리 칼럼을 표시해야 한다면 조인 방식으로 변환하거나 함수, 스칼라 서브쿼리(Scalar Subquery) 등을 사용해야 한다. 조인은 집합간의 곱(Product)의 관계이다. 즉, 1:1 관계의 테이블이 조인하면 1(= 1 * 1) 레벨의 집합이 생성되고, 1:M 관계의 테이블을 조인하면 M(= 1 * M) 레벨의 집합이 생성된다. 그리고 M:N 관계의 테이블을 조인하면 MN(= M * N) 레벨의 집합이 결과로서 생성된다. 예를 들어, 조직(1)과 사원(M) 테이블을 조인하면 결과는 사원 레벨(M)의 집합이 생성된다. 그러나 서브쿼리는 서브쿼리 레벨과는 상관없이 항상 메인쿼리 레벨로 결과 집합이 생성된다. 예를 들어, 메인쿼리로 조직(1), 서브쿼리로 사원(M) 테이블을 사용하면 결과 집합은 조직(1) 레벨이 된다. SQL문에서 서브쿼리 방식을 사용해야 할 때 잘못 판단하여 조인 방식을 사용하는 경우가 있다. 예를 들어, 결과는 조직 레벨이고 사원 테이블에서 체크해야 할 조건이 존재한다고 가정하자. 이런 상황에서 SQL문을 작성할 때 조인을 사용한다면 결과 집합은 사원(M) 레벨이 될 것이다. 이렇게 되면 원하는 결과가 아니기 때문에 SQL문에 DISTINCT를 추가해서 결과를 다시 조직(1) 레벨로 만든다. 이와 같은 상황에서는 조인 방식이 아니라 서브쿼리 방식을 사용해야 한다. 메인쿼리로 조직을 사용하고 서브쿼리로 사원 테이블을 사용하면 결과 집합은 조직 레벨이 되기 때문에 원하는 결과가 된다.

서브쿼리를 사용할 때 다음 사항에 주의해야 한다.

① 서브쿼리를 괄호로 감싸서 사용한다. ② 서브쿼리는 단일 행(Single Row) 또는 복수 행(Multiple Row) 비교 연산자와 함께 사용 가능하다. 단일 행 비교 연산자는 서브쿼리의 결과가 반드시 1건 이하이어야 하고 복수 행 비교 연산자는 서브쿼리의 결과 건수와 상관 없다. ③ 서브쿼리에서는 ORDER BY를 사용하지 못한다. ORDER BY절은 SELECT절에서 오직 한 개만 올 수 있기 때문에 ORDER BY절은 메인쿼리의 마지막 문장에 위치해야 한다.

서브쿼리가 SQL문에서 사용이 가능한 곳은 다음과 같다.

- SELECT 절 - FROM 절 - WHERE 절 - HAVING 절 - ORDER BY 절 - INSERT문의 VALUES 절 - UPDATE문의 SET 절

서브쿼리의 종류는 동작하는 방식이나 반환되는 데이터의 형태에 따라 분류할 수 있다. 동작하는 방식에 따라 서브쿼리를 분류하면 [표 Ⅱ-2-4]와 같이 두 가지로 나눌 수 있다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_215.jpg)

서브쿼리는 메인쿼리 안에 포함된 종속적인 관계이기 때문에 논리적인 실행순서는 항상 메인쿼리에서 읽혀진 데이터에 대해 서브쿼리에서 해당 조건이 만족하지를 확인하는 방식으로 수행되어야 한다. 그러나 실제 서브쿼리의 실행순서는 상황에 따라 달라질 수 있다. 반환되는 데이터의 형태에 따라 서브쿼리는 [표 Ⅱ-2-5]와 같이 세가지로 분류된다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_216.jpg)

#### 1. 단일 행 서브 쿼리

서브쿼리가 단일 행 비교 연산자(=, <, <=, >, >=, <>)와 함께 사용할 때는 서브쿼리의 결과 건수가 반드시 1건 이하이어야 한다. 만약, 서브쿼리의 결과 건수가 2건 이상을 반환하면 SQL문은 실행시간(Run Time) 오류가 발생한다. 이런 종류의 오류는 컴파일 할 때(Compile Time)는 알 수 없는 오류이다. 단일 행 서브쿼리의 예로 '정남일' 선수가 소속된 팀의 선수들에 대한 정보를 표시하는 문제를 가지고 설명해 보면 다음과 같다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_217.jpg)

[그림 Ⅱ-2-13]은 2개의 SQL문으로 구성되어 있다. 정남일 선수의 소속팀을 알아내는 SQL문(서브쿼리 부분)과 이 결과를 이용해서 해당 팀에 소속된 선수들의 정보를 출력하는 SQL문(메인쿼리 부분)으로 구성된다. 정남일 선수가 소속된 팀의 선수들에 대한 정보를 표시하는 문제를 서브쿼리 방식의 SQL문으로 작성하면 다음과 같다.

[예제] SELECT PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버 FROM PLAYER WHERE TEAM_ID = (SELECT TEAM_ID FROM PLAYER WHERE PLAYER_NAME = '정남일') ORDER BY PLAYER_NAME;

[실행 결과] 선수명 포지션 백넘버 ------- ----- ----- 강철 DF 3 김반 MF 14 김영수 MF 30 김정래 GK 33 김창원 DF 5 김회택 TM 꼬레아 FW 16 노병준 MF 22 51개의 행이 선택되었다.

정남일 선수의 소속팀을 알아내는 서브쿼리가 먼저 수행되어 정남일 선수의 소속팀 코드가 반환된다. 메인쿼리는서브쿼리에서 반환된 결과를 이용해서 조건을 만족하는 선수들의 정보를 출력한다. 만약, 정남일 선수가 동명이인이었다면 2건 이상의 결과가 반환되어 SQL문은 오류가 발생될 것이다. 테이블 전체에 하나의 그룹함수를 적용할 때는 그 결과값이 1건이 생성되기 때문에 단일 행 서브쿼리로서 사용 가능하다. 선수들 중에서 키가 평균 이하인 선수들의 정보를 출력하는 문제를 가지고 그룹함수를 사용한 서브쿼리를 알아보도록 한다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_218.jpg)

[그림 Ⅱ-2-14]는 2개의 SQL문으로 구성되어 있다. 선수들의 평균키를 알아내는 SQL문(서브쿼리 부분)과 이 결과를 이용해서 키가 평균 이하의 선수들의 정보를 출력하는 SQL문(메인쿼리 부분)으로 구성된다. [그림 Ⅱ-2-14]를 SQL문으로 작성하면 다음과 같다.

[예제] SELECT PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버 FROM PLAYER WHERE HEIGHT <= (SELECT AVG(HEIGHT) FROM PLAYER) ORDER BY PLAYER_NAME;

[실행 결과] 선수명 포지션 백넘버 ------- ------ ----- 가비 MF 10 강대희 MF 26 강용 DF 2 강정훈 MF 38 강철 DF 3 고규억 DF 29 고민기 FW 24 고종수 MF 22 228개의 행이 선택되었다.

#### 12. 다중 행 서브쿼리

서브쿼리의 결과가 2건 이상 반환될 수 있다면 반드시 다중 행 비교 연산자(IN, ALL, ANY, SOME)와 함께 사용해야 한다. 그렇지 않으면 SQL문은 오류를 반환한다. 다중 행 비교 연산자는 다음과 같다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_219.jpg)

선수들 중에서 ‘정현수’라는 선수가 소속되어 있는 팀 정보를 출력하는 서브쿼리를 작성하면 다음과 같다.

[예제] SELECT REGION_NAME 연고지명, TEAM_NAME 팀명, E_TEAM_NAME 영문팀명 FROM TEAM WHERE TEAM_ID = (SELECT TEAM_ID FROM PLAYER WHERE PLAYER_NAME = '정현수') ORDER BY TEAM_NAME; ORA-01427: 단일 행 하위 질의에 2개 이상의 행이 리턴되었다.

위의 SQL문은 서브쿼리의 결과로 2개 이상의 행이 반환되어 단일 행 비교 연산자인 '='로는 처리가 불가능하기 때문에 에러가 반환되었다. 따라서 다중 행 비교 연산자로 바꾸어서 SQL문을 작성하면 다음과 같다.

[예제] SELECT REGION_NAME 연고지명, TEAM_NAME 팀명, E_TEAM_NAME 영문팀명 FROM TEAM WHERE TEAM_ID IN (SELECT TEAM_ID FROM PLAYER WHERE PLAYER_NAME = '정현수') ORDER BY TEAM_NAME;

[실행 결과] 연고지명 팀명 영문팀명 ------ ----- ----------------------- 전남 드래곤즈 CHUNNAM DRAGONS FC 성남 일화천마 SEONGNAM ILHWA CHUNMA FC 2개의 행이 선택되었다.

실행 결과를 보면 '정현수'란 이름을 가진 선수가 두 명이 존재한다. 소속팀은 각각 전남 드래곤즈팀(K07)과 성남 일화천마팀(K08)이다. 본 예제에서는 동명이인에 대한 내용을 예로 들었지만, 서브쿼리의 실행 결과가 2건 이상이 나오는 모든 경우에 다중 행 비교 연산자를 사용해야 한다.

#### 3. 다중 칼럼 서브쿼리

다중 칼럼 서브쿼리는 서브쿼리의 결과로 여러 개의 칼럼이 반환되어 메인쿼리의 조건과 동시에 비교되는 것을 의미한다. 소속팀별 키가 가장 작은 사람들의 정보를 출력하는 문제를 가지고 다중 칼럼 서브쿼리를 알아보도록 한다. 소속팀별 키가 가장 작은 사람들의 정보는 GROUP BY를 이용하여 찾을 수 있으므로 다음과 같이 SQL문을 작성할 수 있다.

[예제] SELECT TEAM_ID 팀코드, PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE (TEAM_ID, HEIGHT) IN (SELECT TEAM_ID, MIN(HEIGHT) FROM PLAYER GROUP BY TEAM_ID) ORDER BY TEAM_ID, PLAYER_NAME;

[실행 결과] 팀코드 선수명 포지션 백넘버 키 ----- -------- ------ ---- --- K01 마르코스 FW 44 170 K01 박정수 MF 8 170 K02 고창현 MF 8 170 K02 정준 MF 44 170 K03 김중규 MF 42 170 19개의 행이 선택되었다.

SQL문의 실행 결과를 보면 서브쿼리의 결과값으로 소속팀코드(TEAM_ID)와 소속팀별 가장 작은 키를 의미하는 MIN(HEIGHT)라는 두 개의 칼럼을 반환했다. 메인쿼리에서는 조건절에 TEAM_ID와 HEIGHT 칼럼을 괄호로 묶어서 서브쿼리 결과와 비교하여 원하는 결과를 얻었다. 실행 결과에서 보면 하나 팀에서 키가 제일 작은 선수 한 명씩만 반환된 것이 아니라 같은 팀에서 여러 명이 반환된 것을 확인할 수 있다. 이것은 동일 팀 내에서 조건(팀별 가장 작은 키)을 만족하는 선수가 여러 명이 존재하기 때문이다. 그러나 이 기능은 SQL Server에서는 지원되지 않는 기능이다.

#### 4. 연관 서브쿼리

연관 서브쿼리(Correlated Subquery)는 서브쿼리 내에 메인쿼리 칼럼이 사용된 서브쿼리이다. 선수 자신이 속한 팀의 평균 키보다 작은 선수들의 정보를 출력하는 SQL문을 연관 서브쿼리를 이용해서 작성해 보면 다음과 같다

[예제] SELECT T.TEAM_NAME 팀명, M.PLAYER_NAME 선수명, M.POSITION 포지션, M.BACK_NO 백넘버, M.HEIGHT 키 FROM PLAYER M, TEAM T WHERE M.TEAM_ID = T.TEAM_ID AND M.HEIGHT < ( SELECT AVG(S.HEIGHT) FROM PLAYER S WHERE S.TEAM_ID = M.TEAM_ID AND S.HEIGHT IS NOT NULL GROUP BY S.TEAM_ID ) ORDER BY 선수명;

[실행 결과] 팀명 선수명 포지션 백넘버 키 -------- ----- ----- ----- -- 삼성블루윙즈 가비 MF 10 177 삼성블루윙즈 강대희 MF 26 174 스틸러스 강용 DF 2 179 시티즌 강정훈 MF 38 175 드래곤즈 강철 DF 3 178 현대모터스 고관영 MF 32 180 현대모터스 고민기 FW 24 178 삼성블루윙즈 고종수 MF 22 176 224의 행이 선택되었다.

예를 들어, 가비 선수는 삼성블루윙즈팀 소속이므로 삼성블루윙즈팀 소속의 평균키를 구하고 그 평균키와 가비 선수의 키를 비교하여 적을 경우에 선수에 대한 정보를 출력한다. 만약, 평균키 보다 선수의 키가 크거나 같으면 조건에 맞지 않기 때문에 해당 데이터는 출력되지 않는다. 이와 같은 작업을 메인쿼리에 존재하는 모든 행에 대해서 반복 수행한다. EXISTS 서브쿼리는 항상 연관 서브쿼리로 사용된다. 또한 EXISTS 서브쿼리의 특징은 아무리 조건을 만족하는 건이 여러 건이더라도 조건을 만족하는 1건만 찾으면 추가적인 검색을 진행하지 않는다. 다음은 EXISTS 서브쿼리를 사용하여 '20120501' 부터 '20120502' 사이에 경기가 있는 경기장을 조회하는 SQL문이다.

[예제] SELECT STADIUM_ID ID, STADIUM_NAME 경기장명 FROM STADIUM A WHERE EXISTS (SELECT 1 FROM SCHEDULE X WHERE X.STADIUM_ID = A.STADIUM_ID AND X.SCHE_DATE BETWEEN '20120501' AND '20120502')

[실행 결과] ID 경기장명 --- --------------------------------- B01 인천월드컵경기장 B04 수원월드컵경기장 B05 서울월드컵경기장 C02 부산아시아드경기장 4개의 행이 선택되었다.

#### 5. 그밖에 위치에서 사용하는 서브쿼리

##### 가. SELECT 절에 서브쿼리 사용하기

다음은 SELECT 절에서 사용하는 서브쿼리인 스칼라 서브쿼리(Scalar Subquery)에 대해서 알아본다. 스칼라 서브쿼리는 한 행, 한 칼럼(1 Row 1 Column)만을 반환하는 서브쿼리를 말한다. 스칼라 서브쿼리는 칼럼을 쓸 수 있는 대부분의 곳에서 사용할 수 있다. 선수 정보와 해당 선수가 속한 팀의 평균 키를 함께 출력하는 예제로 스칼라 서브쿼리를 설명하면 다음과 같다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_220.jpg)

[그림 Ⅱ-2-15]는 2개의 SQL문으로 구성되어 있다. 선수들의 정보를 출력하는 SQL문(메인쿼리 부분)과 해당 선수의 소속팀별 평균키를 알아내는 SQL문(서브쿼리 부분)으로 구성된다. 여기서 선수의 소속팀별 평균키를 알아내는 스칼라 서브쿼리는 메인쿼리의 결과 건수만큼 반복수행 된다. [그림 Ⅱ-2-15]를 SQL문으로 작성하면 다음과 같다.

[예제] SELECT PLAYER_NAME 선수명, HEIGHT 키, (SELECT AVG(HEIGHT) FROM PLAYER X WHERE X.TEAM_ID = P.TEAM_ID) 팀평균키 FROM PLAYER P

[실행 결과] 선수명 키 팀평균키 ------- ---- ------------- 가비 177 179.067 가이모토 182 178.854 강대희 174 179.067 강성일 182 177.485 강용 179 179.911 강정훈 175 177.485 강철 178 178.391 고관영 180 180.422 480개의 행이 선택되었다.

스칼라 서브쿼리 또한 단일 행 서브쿼리이기 때문에 결과가 2건 이상 반환되면 SQL문은 오류를 반환한다.

##### 나. FROM 절에서 서브쿼리 사용하기

FROM 절에서 사용되는 서브쿼리를 인라인 뷰(Inline View)라고 한다. FROM 절에는 테이블 명이 오도록 되어 있다. 그런데 서브쿼리가 FROM 절에 사용되면 어떻게 될까? 서브쿼리의 결과가 마치 실행 시에 동적으로 생성된 테이블인 것처럼 사용할 수 있다. 인라인 뷰는 SQL문이 실행될 때만 임시적으로 생성되는 동적인 뷰이기 때문에 데이터베이스에 해당 정보가 저장되지 않는다. 그래서 일반적인 뷰를 정적 뷰(Static View)라고 하고 인라인 뷰를 동적 뷰(Dynamic View)라고도 한다. 뷰에 대해서는 뒤에서 좀더 설명하기로 한다. 인라인 뷰는 테이블 명이 올 수 있는 곳에서 사용할 수 있다. 서브쿼리의 칼럼은 메인쿼리에서 사용할 수 없다고 했다. 그러나 인라인 뷰는 동적으로 생성된 테이블이다. 인라인 뷰를 사용하는 것은 조인 방식을 사용하는 것과 같다. 그렇기 때문에 인라인 뷰의 칼럼은 SQL문 자유롭게 참조할 수 있다. K-리그 선수들 중에서 포지션이 미드필더(MF)인 선수들의 소속팀명 및 선수 정보를 출력하고자 한다. 인라인 뷰를 활용해서 SQL문을 만들어 보자.

[예제] SELECT T.TEAM_NAME 팀명, P.PLAYER_NAME 선수명, P.BACK_NO 백넘버 FROM (SELECT TEAM_ID, PLAYER_NAME, BACK_NO FROM PLAYER WHERE POSITION = 'MF') P, TEAM T WHERE P.TEAM_ID = T.TEAM_ID ORDER BY 선수명;

[실행 결과] 팀명 선수명 백넘버 --------- ------- ----- 삼성블루윙즈 가비 10 삼성블루윙즈 강대희 26 시티즌 강정훈 38 현대모터스 고관영 32 삼성블루윙즈 고종수 22 삼성블루윙즈 고창현 8 시티즌 공오균 22 일화천마 곽치국 32 162개의 행이 선택되었다.

SQL문을 보면 선수들 중에서 포지션이 미드필더(MF) 선수들을 인라인 뷰를 통해서 추출하고 인라인 뷰의 결과와 TEAM 테이블과 조인해서 팀명(TEAM_NAME)을 출력하고 있다. 인라인 뷰에서는 ORDER BY절을 사용할 수 있다. 인라인 뷰에 먼저 정렬을 수행하고 정렬된 결과 중에서 일부 데이터를 추출하는 것을 TOP-N 쿼리라고 한다. TOP-N 쿼리를 수행하기 위해서는 정렬 작업과 정렬 결과 중에서 일부 데이터만을 추출할 수 있는 방법이 필요하다. Oracle에서는 ROWNUM이라는 연산자를 통해서 결과로 추출하고자 하는 데이터 건수를 제약할 수 있다.

[예제] Oracle SELECT PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM (SELECT PLAYER_NAME, POSITION, BACK_NO, HEIGHT FROM PLAYER WHERE HEIGHT IS NOT NULL ORDER BY HEIGHT DESC) WHERE ROWNUM <= 5;

[예제] SQL Server SELECT TOP(5) PLAYER_NAME AS 선수명, POSITION AS 포지션, BACK_NO AS 백넘버, HEIGHT AS 키 FROM PLAYER WHERE HEIGHT IS NOT NULL ORDER BY HEIGHT DESC

[실행 결과] 선수명 포지션 백넘버 키 -------- ----- --- --- 서동명 GK 21 196 권정혁 GK 1 195 김석 FW 20 194 정경두 GK 41 194 이현 GK 1 192 5개의 행이 선택되었다.

당 SQL문의 인라인 뷰에서 선수의 키를 내림차순으로 정렬(가장 키가 큰 선수부터 출력)한 후 메인쿼리에서 ROWNUM을 사용해서 5명의 선수의 정보만을 추출하였다. 이것은 모든 선수들 중에서 가장 키가 큰 5명의 선수를 출력한 것이다. 만약, 다른 선수 중에서 키가 192인 선수가 더 존재하더라도 해당 SQL문에서는 데이터가 출력되지 않는다. 이런 데이터까지 추출하고자 한다면 분석함수의 RANK관련 함수를 사용해야 한다.

##### 다. HAVING 절에서 서브쿼리 사용하기

HAVING 절은 그룹함수와 함께 사용될 때 그룹핑된 결과에 대해 부가적인 조건을 주기 위해서 사용한다. 평균키가 삼성 블루윙즈팀의 평균키보다 작은 팀의 이름과 해당 팀의 평균키를 구하는 SQL문을 작성하면 다음과 같다.

[예제] SELECT P.TEAM_ID 팀코드, T.TEAM_NAME 팀명, AVG(P.HEIGHT) 평균키 FROM PLAYER P, TEAM T WHERE P.TEAM_ID = T.TEAM_ID GROUP BY P.TEAM_ID, T.TEAM_NAME HAVING AVG(P.HEIGHT) < (SELECT AVG(HEIGHT) FROM PLAYER WHERE TEAM_ID ='K02')

[실행 결과] 팀코드 팀명 평균키 ---- ----------- ------ K13 강원FC 173.667 K15 대구FC 175.333 K11 경남FC 176.333 K14 제주유나이티드FC 169.5 K12 광주상무 173.5 K07 드래곤즈 178.391 K08 일화천마 178.854 K10 시티즌 177.485 8개의 행이 선택되었다.

##### 라. UPDATE문의 SET 절에서 사용하기

현재 TEAM 테이블에는 STADIUM_NAME 칼럼이 없다. TEAM 테이블에 STADIUM_NAME을 추가(ALTER TABLE ADD COLUMN)하였다고 가정하자. TEAM 테이블에 추가된 STADIUM_NAME의 값을 STADIUM 테이블을 이용하여 변경하고자 할 때 다음과 같이 SQL문을 작성할 수 있다.

UPDATE TEAM A SET A.STADIUM_NAME = (SELECT X.STADIUM_NAME FROM STADIUM X WHERE X.STADIUM_ID = A.STADIUM_ID);

서브쿼리를 사용한 변경 작업을 할 때 서브쿼리의 결과가 NULL을 반환할 경우 해당 컬럼의 결과가 NULL이 될 수 있기 때문에 주의해야 한다.

##### 마. INSERT문의 VALUES절에서 사용하기

PLAYER 테이블에 '홍길동'이라는 선수를 삽입하고자 한다. 이때 PLAYER_ID의 값을 현재 사용중인 PLAYER_ID에 1을 더한 값으로 넣고자 한다. 다음과 같이 SQL문을 SQL문을 작성할 수 있다.

INSERT INTO PLAYER(PLAYER_ID, PLAYER_NAME, TEAM_ID) VALUES((SELECT TO_CHAR(MAX(TO_NUMBER(PLAYER_ID))+1) FROM PLAYER), '홍길동', 'K06');

#### 6. 뷰(View)

테이블은 실제로 데이터를 가지고 있는 반면, 뷰(View)는 실제 데이터를 가지고 있지 않다. 뷰는 단지 뷰 정의(View Definition)만을 가지고 있다. 질의에서 뷰가 사용되면 뷰 정의를 참조해서 DBMS 내부적으로 질의를 재작성(Rewrite)하여 질의를 수행한다. 뷰는 실제 데이터를 가지고 있지 않지만 테이블이 수행하는 역할을 수행하기 때문에 가상 테이블(Virtual Table)이라고도 한다. 뷰는 [표 Ⅱ-2-7]과 같은 장점을 갖는다.

![sql가이드](http://www.dbguide.net/publishing/img/knowledge/SQL_221.jpg)

뷰는 다음과 같이 CREATE VIEW문을 통해서 생성할 수 있다.

CREATE VIEW V_PLAYER_TEAM AS SELECT P.PLAYER_NAME, P.POSITION, P.BACK_NO, P.TEAM_ID, T.TEAM_NAME FROM PLAYER P, TEAM T WHERE P.TEAM_ID = T.TEAM_ID;

해당 뷰는 선수 정보와 해당 선수가 속한 팀명을 함께 추출하는 것이다. 뷰의 명칭은 'V_PLAYER_TEAM'이다. 뷰는 테이블뿐만 아니라 이미 존재하는 뷰를 참조해서도 생성할 수 있다.

CREATE VIEW V_PLAYER_TEAM_FILTER AS SELECT PLAYER_NAME, POSITION, BACK_NO, TEAM_NAME FROM V_PLAYER_TEAM WHERE POSITION IN ('GK', 'MF');

V_PLAYER_TEAM_FILTER 뷰는 이미 앞에서 생성했던 V_PLAYER_TEAM 뷰를 기반으로 해서 생성된 뷰다. V_PLAYER_TEAM_FILTER 뷰는 선수 포지션이 골키퍼(GK), 미드필더(MF)인 선수만을 추출하고자 하는 뷰이다.(뷰를 포함하는 뷰를 잘못 생성하는 경우 성능상의 문제를 유발할 수 있으므로, 뷰와 SQL의 수행원리를 잘 이해하고 사용하기 바란다) 뷰를 사용하기 위해서는 해당 뷰의 이름을 이용하면 된다. 뷰를 사용하는 방법은 다음과 같다.

[예제] SELECT PLAYER_NAME, POSITION, BACK_NO, TEAM_ID, TEAM_NAME FROM V_PLAYER_TEAM WHERE PLAYER_NAME LIKE '황%'

[실행 결과] PLAYER_NAME POSITION BACK_NO TEAM ID TEAM_NAME ----------- ------- ------- ------- --------- 황철민 MF 35 K06 아이파크 황승주 DF 98 K05 현대모터스 황연석 FW 16 K08 일화천마 3개의 행이 선택되었다.

이것은 V_PLAYER_TEAM 뷰에서 성이 '황'씨인 선수만을 추출하는 SQL문이다. 결과로서 3건이 추출되었다. 뷰를 사용하는 경우에는 DBMS가 내부적으로 SQL문을 다음과 같이 재작성한다.

SELECT PLAYER_NAME, POSITION, BACK_NO, TEAM_ID, TEAM_NAME FROM (SELECT P.PLAYER_NAME, P.POSITION, P.BACK_NO, P.TEAM_ID, T.TEAM_NAME FROM PLAYER P, TEAM T WHERE P.TEAM_ID = T.TEAM_ID) WHERE PLAYER_NAME LIKE '황%'

이것은 앞에서 설명했던 인라인 뷰와 유사한 모습임을 알 수 있다. 이와 같은 형태로 사용되기 때문에 뷰는 데이터를 저장하지 않고도 데이터를 조회할 수 있다. 뷰를 제거하기 위해서는 DROP VIEW문을 사용한다.

DROP VIEW V_PLAYER_TEAM; DROP VIEW V_PLAYER_TEAM_FILTER;
