- `Confluence`에 테이블 컬럼 명세 정리가 필요했음
- 쿼리 통해 전체 테이블 개수 확인 
	- 백업 DB로 보이는 테이블은 필터링 해서 처리
- 모니터 3개 역할 
	- `왼쪽` : Dbeaver
	- `가운데` : Google Sheet, 기록 및 중간 변환
		- Dbeaver에서 컬럼, 데이터 타입 복사시 바로 Confluence 붙여 넣기 하면 배경색이 지워지지 않음 
		- 그래서 Google Sheet 통해 배경색을 흰색으로 바꾸고, 데이터 타입은 `대문자(=upper)` 변환하여 해결
	- `오른쪽 (노트북)` : Confluence 문서 복제 및 정리


```sql
/*
테이블 개수 (시스템 테이블 제외) : 255
*/
select count(*)
from sys.tables
where is_ms_shipped = 0;


/*
 테이블별 컬럼 개수 (시스템 테이블 제외, 조건식)
 151개 (뒤에 숫자 붙은거 백업이라 인식하여 제외)
*/
SELECT
    t.name AS TableName,
    COUNT(c.column_id) AS TotalColumns
FROM
    sys.tables t
INNER JOIN
    sys.columns c ON t.object_id = c.object_id
WHERE
    t.is_ms_shipped = 0  -- 시스템 테이블 제외
    AND t.name NOT LIKE '%_[0-9]%'
GROUP BY
    t.name
ORDER BY
    TableName;
    
/*
	테이블별 
	- 컬럼명
	- 데이터타입 
	- 사이즈
	- nullable 여부
*/
SELECT
    T.TABLE_NAME,
    C.COLUMN_NAME,
    C.DATA_TYPE,
    C.CHARACTER_MAXIMUM_LENGTH, -- 이 필드가 nvarchar(10)의 실제 문자열 길이를 나타냄 (바이트 아님)
    C.IS_NULLABLE
    -- PK 여부는 INFORMATION_SCHEMA에서 직접 확인하기 어려워 별도 쿼리가 필요함
FROM
    INFORMATION_SCHEMA.COLUMNS C
JOIN
    INFORMATION_SCHEMA.TABLES T ON C.TABLE_NAME = T.TABLE_NAME AND T.TABLE_TYPE = 'BASE TABLE'
WHERE
    T.TABLE_NAME = N'CCTV_MANAGER'
ORDER BY
    C.ORDINAL_POSITION;
```

