[TOC]

# 1.题目185 部门工资前三高的所有员工

## 1.1 题目描述

https://leetcode-cn.com/problems/department-top-three-salaries/

![image-20210424083915785](images/image-20210424083915785.png)

## 1.2 解答

```sql
select t.*, t2.Name
from (
         select DepartmentId,
                id,
                name,
                Salary,
                dense_rank() over (partition by DepartmentId order by Salary desc ) as rn

         from lc_185_employee
     ) t
         left join lc_185_department t2
                   on t.DepartmentId = t2.Id
where t.rn <= 3;
```





# 2. 题目262 行程和用户



## 2.1 题目描述

https://leetcode-cn.com/problems/trips-and-users/

![image-20210424103018955](images/image-20210424103018955.png)

![image-20210424103035618](images/image-20210424103035618.png)

![image-20210424103050052](images/image-20210424103050052.png)

![image-20210424103100187](images/image-20210424103100187.png)

## 2.2 解答

```sql
select t.Request_at as Day,
       round(
               sum(if(t.Status = 'completed', 0, 1)) / count(t.Status)
           , 2
           )        as `Cancellation Rate`


from lc_262_tripes t
         join lc_262_users t1 on (t.Client_Id = t1.users_id)
         join lc_262_users t2 on (t.Client_Id = t2.users_id)
where to_date(t.request_at) between '2013-10-01' and '2013-10-03'
group by Request_at;
```





# 3. 题目569 员工薪水中位数

## 3.1 题目描述

https://leetcode-cn.com/problems/median-employee-salary/

![image-20210424103937326](images/image-20210424103937326.png)

## 3.2 解答

```sql
select Id, Company, Salary
from (
         select Id,
                Company,
                Salary,
                row_number() over (partition by Company order by Salary) as rnk,
                count(Salary) over (partition by Company)                as cnt
         from lc_569_employee
     ) t
where rnk in (cnt / 2, cnt / 2 + 1, cnt / 2 + 0.5);
```





# 4. 题目571 给定数字的频率查询中位数

## 4.1 题目描述

![image-20210424105941620](images/image-20210424105941620.png)

## 4.2 解答

中位数核心思想：按照asc排序，再按desc排序，amount都要大于等于总数/2

```sql
select avg(number) as median
from (select number,
             sum(frequency) over (order by number)  as asc_amount,
             sum(frequency) over (order by number desc) as desc_amount,
             sum(frequency) over ()                     as total_num
      from lc_571_numbers) a
where asc_amount >= total_num / 2
  and desc_amount >= total_num / 2;
```







# 5. 题目579 查询员工的累计薪水

## 5.1 题目描述

https://leetcode-cn.com/problems/find-cumulative-salary-of-an-employee/

![image-20210424140004846](images/image-20210424140004846.png)

![image-20210424140018130](images/image-20210424140018130.png)

![image-20210424140032577](images/image-20210424140032577.png)



## 5.2 题目解答

```sql
SELECT Id, Month, Salary
FROM (SELECT Id,
             Month,
             SUM(Salary) OVER (PARTITION BY Id ORDER BY Month ROWS 2 PRECEDING) AS Salary,
             rank() OVER (PARTITION BY Id ORDER BY Month DESC)                  AS r
      FROM lc_579_employee) t
WHERE r > 1
ORDER BY Id, Month DESC;
```



# 6. 题目601 体育馆的人流量

## 6.1 题目描述

https://leetcode-cn.com/problems/human-traffic-of-stadium/

![image-20210424141915543](images/image-20210424141915543.png)

![image-20210424141928379](images/image-20210424141928379.png)

## 6.2 解答

```sql
with t60101 as (
    select id,
           visit_date,
           people,
           id - rank() over (order by id) as rn
    from lc_601_stadium
    where people >= 100
)

select id,
       visit_date,
       people
from t60101
where rn in (
    select rn
    from t60101
    group by rn
    having count(1) >= 3);
```





# 7. 题目615 平均工资：部门与公司比较

## 7.1 题目描述

![image-20210424162017296](images/image-20210424162017296.png)

![image-20210424162029780](images/image-20210424162029780.png)

## 7.2 解答

```sql
with t61501 as (
    select sal.employee_id,
           amount,
           pay_date,
           emp.department_id as dep_id
    from lc_615_salary sal
             left join lc_615_employee emp
                       on sal.employee_id = emp.employee_id
)
select substr(pay_date, 1, 7)                                                                          as pay_month,
       dep_id                                                                                          as department_id,
       (case when dep_avg > com_avg then 'higher' when dep_avg < com_avg then 'lower' else 'same' end) as comparision
from (
         select pay_date,
                dep_id,
                round(avg(amount) over (partition by dep_id,month(pay_date)), 2) as dep_avg,
                round(avg(amount) over (partition by month(pay_date)), 2)        as com_avg
         from t61501
     ) t
group by dep_id, substr(pay_date, 1, 7),
         (case when dep_avg > com_avg then 'higher' when dep_avg < com_avg then 'lower' else 'same' end)
order by pay_month desc
```







# 8. 题目618 学生地理信息报告

## 8.1 题目描述

![image-20210424162525016](images/image-20210424162525016.png)

## 8.2 解答

```sql
select 
max(case when continent='America' then name end )as America,
max(case when continent='Asia' then name end )as Asia,
max(case when continent='Europe' then name end) as Europe
from (select  name,continent,row_number() over(partition by continent order by name) rk
from student)t
group by rk
```





# 9. 题目1097 游戏玩法分析V

## 9.1 题目描述

![image-20210424165654072](images/image-20210424165654072.png)

![image-20210424165706941](images/image-20210424165706941.png)

## 9.2 解答

```sql
SELECT first_day                 as install_dt,
       count(DISTINCT player_id) as installs,
       ROUND(
                   (SUM(if(datediff(event_date, first_day) = 1, 1, 0))) / (count(DISTINCT player_id)), 2
           )                     as Day1_retention
FROM (
         SELECT player_id,
                event_date,
                MIN(event_date) over (partition by player_id) as first_day
         FROM lc_1097_activity
     ) a
GROUP BY first_day;
```



# 10. 题目1127 用户购买平台

## 10.1 题目描述

![image-20210424203946361](images/image-20210424203946361.png)

![image-20210424203957936](images/image-20210424203957936.png)

## 10.2 解答

```sql
select a.spend_date,
       a.platform,
       nvl(b.total_amount, 0) as total_amount,
       nvl(b.total_users, 0)  as total_users
from (select distinct a.spend_date, b.platform
      from lc_1127_spending a,
           (select 'mobile' as platform union all select 'desktop' union all select 'both') b
      order by a.spend_date) a
         left join
     (select spend_date,
             platform,
             sum(total_amount) as total_amount,
             count(user_id)    as total_users
      from (select case
                       when sum(if(platform = 'mobile', 1, 0)) >= 1 and sum(if(platform = 'desktop', 1, 0)) >= 1
                           then 'both'
                       when sum(if(platform = 'mobile', 1, 0)) >= 1 and sum(if(platform = 'desktop', 1, 0)) = 0
                           then 'mobile'
                       else 'desktop' end platform,
                   spend_date,
                   user_id,
                   sum(amount) as         total_amount
            from lc_1127_spending
            group by spend_date, user_id) a
      group by spend_date, platform) b
     on a.spend_date = b.spend_date and a.platform = b.platform;
```

https://leetcode-cn.com/problems/user-purchase-platform/comments/