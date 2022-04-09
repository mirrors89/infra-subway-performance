<p align="center">
    <img width="200px;" src="https://raw.githubusercontent.com/woowacourse/atdd-subway-admin-frontend/master/images/main_logo.png"/>
</p>
<p align="center">
  <img alt="npm" src="https://img.shields.io/badge/npm-%3E%3D%205.5.0-blue">
  <img alt="node" src="https://img.shields.io/badge/node-%3E%3D%209.3.0-blue">
  <a href="https://edu.nextstep.camp/c/R89PYi5H" alt="nextstep atdd">
    <img alt="Website" src="https://img.shields.io/website?url=https%3A%2F%2Fedu.nextstep.camp%2Fc%2FR89PYi5H">
  </a>
  <img alt="GitHub" src="https://img.shields.io/github/license/next-step/atdd-subway-service">
</p>

<br>

# 인프라공방 샘플 서비스 - 지하철 노선도

<br>

## 🚀 Getting Started

### Install
#### npm 설치
```
cd frontend
npm install
```
> `frontend` 디렉토리에서 수행해야 합니다.

### Usage
#### webpack server 구동
```
npm run dev
```
#### application 구동
```
./gradlew clean build
```
<br>

## 미션

* 미션 진행 후에 아래 질문의 답을 작성하여 PR을 보내주세요.

### 1단계 - 쿼리 최적화

1. 인덱스 설정을 추가하지 않고 아래 요구사항에 대해 1s 이하(M1의 경우 2s)로 반환하도록 쿼리를 작성하세요.

- 활동중인(Active) 부서의 현재 부서관리자 중 연봉 상위 5위안에 드는 사람들이 최근에 각 지역별로 언제 퇴실했는지 조회해보세요. (사원번호, 이름, 연봉, 직급명, 지역, 입출입구분, 입출입시간)


- **M1 노트북으로 실행한 결과입니다.**
```sql
SELECT 
	SUB_TABLE.사원번호,
	SUB_TABLE.이름,
	SUB_TABLE.연봉,
	SUB_TABLE.직급명,
	사원출입기록.입출입시간,
	사원출입기록.지역, 
	사원출입기록.입출입구분
FROM (
	SELECT 
		사원.사원번호, 
		사원.이름, 
		급여.연봉, 
		직급.직급명
	FROM 사원
	INNER JOIN 직급 ON 사원.사원번호 = 직급.사원번호
	INNER JOIN 급여 ON 사원.사원번호 = 급여.사원번호
	INNER JOIN 부서관리자 ON 사원.사원번호 = 부서관리자.사원번호
	INNER JOIN 부서 ON 부서.부서번호 = 부서관리자.부서번호
	WHERE 부서.비고 = 'active' 
	AND 직급.종료일자 > now() 
	AND 급여.종료일자 > now()
	AND 부서관리자.종료일자 > now()
	ORDER BY 급여.연봉 DESC
	LIMIT 5
) SUB_TABLE
INNER JOIN 사원출입기록 ON SUB_TABLE.사원번호 = 사원출입기록.사원번호 
WHERE 사원출입기록.입출입구분 = 'O';
```
![1주차미션결과.png](result/1주차미션결과.png)



---

### 2단계 - 인덱스 설계

1. 인덱스 적용해보기 실습을 진행해본 과정을 공유해주세요
-  Coding as a Hobby 와 같은 결과를 반환하세요.
    ```sql
    CREATE INDEX `idx_hobby` ON `subway`.`programmer` (hobby);
     
    SELECT 
        hobby,
        ROUND((count(*) * 100) / (select count(*) f![img.png](img.png)rom subway.programmer), 1) as percent
    FROM subway.programmer
    GROUP BY hobby
    ORDER BY percent desc;
    ```
    ![result/2단계_1번.png](result/2단계_1번.png)


-  프로그래머별로 해당하는 병원 이름을 반환하세요. (covid.id, hospital.name)
    ```sql
    ALTER TABLE `hospital` ADD PRIMARY KEY (id);
    ALTER TABLE `programmer` ADD PRIMARY KEY (id);
    CREATE INDEX `idx_programmer_id` ON `subway`.`covid` (programmer_id);

    SELECT 
        covid.id, 
        hospital.name, 
        programmer.id
    FROM programmer
    INNER JOIN covid ON covid.programmer_id = programmer.id
    INNER JOIN hospital ON covid.hospital_id = hospital.id;
    ```
   ![result/2단계_2번.png](result/2단계_2번.png)

- 프로그래밍이 취미인 학생 혹은 주니어(0-2년)들이 다닌 병원 이름을 반환하고 user.id 기준으로 정렬하세요. (covid.id, hospital.name, user.Hobby, user.DevType, user.YearsCoding)
    ```sql
    SELECT
        covid.id, 
        hospital.name, 
        user.hobby, 
        user.dev_type, 
        user.years_coding
    FROM programmer user
    INNER JOIN covid ON covid.programmer_id = user.id
    INNER JOIN hospital ON covid.hospital_id = hospital.id
    WHERE user.hobby = 'Yes'
    AND (years_coding_prof = '0-2 years' OR years_coding_prof = 'NA');
    ```
    ![result/2단계_3번.png](result/2단계_3번.png)


- 서울대병원에 다닌 20대 India 환자들을 병원에 머문 기간별로 집계하세요. (covid.Stay)
    ```sql
    ALTER TABLE `member` ADD PRIMARY KEY (id);
    CREATE INDEX `idx_age` ON `subway`.`member` (age);
    CREATE INDEX `idx_hospital_id` ON `subway`.`covid` (hospital_id);
    CREATE INDEX `idx_member_id` ON `subway`.`programmer` (member_id);
    CREATE INDEX `idx_country` ON `subway`.`programmer` (country);
    CREATE INDEX `idx_name` ON `subway`.`hospital` (name);

    SELECT 
      covid.stay,
        count(covid.stay)
    FROM member
    INNER JOIN programmer ON member.id = programmer.member_id
    INNER JOIN covid ON covid.member_id = member.id
    INNER JOIN hospital ON covid.hospital_id = hospital.id
    WHERE hospital.name = '서울대병원'
    AND member.age BETWEEN 20 AND 29
    AND programmer.country =  'India'
    GROUP BY covid.stay;
    ```
    ![result/2단계_4번.png](result/2단계_4번.png)


- 서울대병원에 다닌 30대 환자들을 운동 횟수별로 집계하세요. (user.Exercise)
    ```sql
    SELECT
        programmer.exercise,
        count(programmer.exercise)
    FROM member
    INNER JOIN programmer ON member.id = programmer.member_id
    INNER JOIN covid ON covid.member_id = member.id
    INNER JOIN hospital ON covid.hospital_id = hospital.id
    WHERE hospital.name = '서울대병원'
    AND member.age BETWEEN 30 AND 39
    GROUP BY programmer.exercise; 
    ```
   ![result/2단계_4번.png](result/2단계_4번.png)


---

### 추가 미션

1. 페이징 쿼리를 적용한 API endpoint를 알려주세요
