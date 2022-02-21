문제 링크:https://programmers.co.kr/learn/courses/30/lessons/59414  
ANIMAL_INS 테이블에 등록된 모든 레코드에 대해, 각 동물의 아이디와 이름, 들어온 날짜1를 조회하는 SQL문을 작성해주세요.  
이때 결과는 아이디 순으로 조회해야 합니다.
# 풀이

```sql
SELECT ANIMAL_ID, NAME, DATE_FORMAT (DATETIME,'%Y-%m-%d') AS '날짜'
FROM ANIMAL_INS
ORDER BY ANIMAL_ID
```
