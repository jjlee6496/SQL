# JOIN
다른 정보가 들어있는 두개 이상의 테이블과 연결 또는 결합하여 데이터를 출력하는 것을 JOIN이라고 한다.  
* 일반적인 경우 행들은 PRIMARY KEY(PK)나 FOREIGN KEY(FK) 값의 연관에 의해 JOIN이 성립된다. 하지만 어떤 경우에는 이러한 PK, FK의 관계가 없어도 논리적인 값들의 연관만으로 JOIN이 성립 가능하다.  
* SQL에서 데이터를 처리할 때는 단 두 개의 집합 간에만 조인이 일어난다는 것이다. 예를 들어 A, B, C, D 4개의 테이블을 조인하고자 할 경우 옵티마이저는 ( ( (A JOIN D) JOIN C) JOIN B)와 같이 순차적으로 조인을 처리하게 된다. 이 때문에 (equi)join조건에 테이블 개수 -1 개의 조건이 필요함. 이때 테이블의 조인 순서는 옵티마이저에 의해서 결정되고 과목3의 주요 튜닝 포인트가 된다.  

## EQUI JOIN
* EQUI(등가) JOIN은 두 개의 테이블 간에 칼럼 값들이 서로 정확하게 일치하는 경우에 사용되는 방법, JOIN의 조건은 WHERE 절에 기술하게 되는데 “=” 연산자를 사용해서 표현한다.  
  
[예제] 선수 테이블과 팀 테이블에서 선수 이름과 소속된 팀의 이름을 출력하시오.
![image](https://user-images.githubusercontent.com/96418287/155869747-b615399d-bb2c-4998-a729-d7493b4e08c9.png)

   

```sql
SELECT P.PLAYER_NAME 선수명, T.TEAM_NAME 소속팀명
FROM PLAYER P, TEAM T
WHERE P.TEAM_ID = T.TEAM_ID;
# 또는 INNER JOIN 명시
SELECT P.PLAYER_NAME 선수명, T.TEAM_NAME 소속팀명
FROM PLAYER P INNER JOIN TEAM T
ON P.TEAM_ID = T.TEAM_ID;
```
   
![image](https://user-images.githubusercontent.com/96418287/155869773-6a67d5be-0d17-4988-9075-b94d666a5742.png)

[그림 Ⅱ-1-14]의 실바 선수를 예로 들면 백넘버는 45번이고, 소속팀코드는 K07번이다. K07번 팀코드의 팀명은 드래곤즈이고 연고지는 전남이라는 결과를 얻을 수 있게 된다.
[예제] [그림 Ⅱ-1-14]의 데이터를 출력하기 위한 SELECT SQL 문장을 작성한다.
```sql
SELECT P.PLAYER_NAME 선수명,P.BACK_NO 백넘버,P.TEAM_ID 소속팀코드, T.TEAM_NAME 소속팀명,T.REGION_NAME 연고지
FROM PLAYER P, TEAM T
WHERE P.TEAM_ID = T.TEAM_ID;
# 또는 INNER JOIN 명시
SELECT P.PLAYER_NAME 선수명,P.BACK_NO 백넘버,P.TEAM_ID 소속팀코드, T.TEAM_NAME 소속팀명,T.REGION_NAME 연고지
FROM PLAYER P INNER JOIN TEAM T
ON P.TEAM_ID = T.TEAM_ID;
```
## NON-EQUI JOIN
Non EQUI(비등가) JOIN은 두 개의 테이블 간에 칼럼 값들이 서로 정확하게 일치하지 않는 경우에 사용된다. Non EQUI JOIN의 경우에는 “=” 연산자가 아닌 다른(Between, >, >=, <, <= 등) 연산자들을 사용하여 JOIN을 수행하는 것이다. 데이터 모델에 따라서 Non EQUI JOIN이 불가능한 경우도 있다.   
 ![image](https://user-images.githubusercontent.com/96418287/155869794-69e15962-e78f-4b20-9733-3042e8cd589d.png)

테이블 간의 관계를 설명하기 위해 먼저 사원(EMP) 테이블과 급여등급(SALGRADE) 테이블에 있는 데이터와 이들 간의 관계를 나타내는 [그림 Ⅱ-1-16]을 가지고 실제적인 데이터들이 어떻게 연결되는지 설명한다. 급여등급(SALGRADE) 테이블에는 1급(700 이상 ~ 1200 이하), 2급(1201 이상 ~ 1400 이하), 3급(1401 이상 ~ 2000 이하), 4급(2001 이상 ~ 3000 이하), 5급(3001 이상 ~ 9999 이하)으로 구분한 5개의 급여등급이 존재한다고 가정한다.
[예제] 사원 14명 모두에 대해 아래 SQL로 급여와 급여등급을 알아본다.
```sql
SELECT E.ENAME 사원명, E.SAL 급여, S.GRADE 급여등급
FROM EMP E, SALGRADE S
WHERE E.SAL BETWEEN S.LOSAL AND S.HISAL
```
# 3개 이상 TABLE JOIN
![image](https://user-images.githubusercontent.com/96418287/155869803-a0b29f16-617b-4eed-871a-1c30c4fb9a99.png)


[예제]선수들 별로 포지션 연고지 팀명 홈그라운드 경기장이 어디인지를 출력
```sql
SELECT P.PLAYER_NAME 선수명, P.POSITION 포지션, T.REGION_NAME 연고지, T.TEAM_NAME 팀명,  S.STADIUM_ID 구장명
FROM PLAYER P, TEAM T, STADIUM S
WHERE P.TEAM_ID = T.TEAM_ID AND T.STADIUM_ID = S.STADIUM_ID
ORDER BY 선수명;
#또는 INNER JOIN 명시
SELECT P.PLAYER_NAME 선수명, P.POSITION 포지션, T.REGION_NAME 연고지, T.TEAM_NAME 팀명,  S.STADIUM_ID 구장명
FROM PLAYER P INNER JOIN TEAM T
ON P.TEAM_ID = T.TEAM_ID INNER JOIN STADIUM S
ON T.STADIUM_ID = S.STADIUM_ID
ORDER BY 선수명;
```
# 출처
https://dataonair.or.kr/db-tech-reference/d-guide/sql/?pageid=3&mod=document&uid=345
