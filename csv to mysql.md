# csv to mysql

```sql
SET GLOBAL local_infile=1;
SHOW GLOBAL VARIABLES LIKE 'local_infile';

LOAD DATA LOCAL INFILE '/home/plant_data.csv' INTO TABLE plant        
CHARACTER SET UTF8
FIELDS 
	TERMINATED BY '|'
	OPTIONALLY ENCLOSED BY '"' 
IGNORE 1 ROWS;
```



### 에러 발생

```bash
LOAD DATA LOCAL INFILE file request rejected due to restrictions on access.
```

mysql client와 서버측 모두 local_infile을 1로 바꾸어 주어야한다.

```bash
# mysql client
mysql --local_infile -u root -p

# server 
SET GLOBAL local_infile=1;

```



