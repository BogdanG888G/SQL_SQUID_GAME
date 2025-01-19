# Задачи

## 1 задание

```sql
select * from player
where status = 'alive' and debt > 400000000 and (age > 65 or (vice = 'Gambling' and has_close_family = 'false')) 
              
              
```

## 2 задание:

```sql

                with temp_count as (
select count(*) from player
where status = 'alive' and isinsider = 'false'
)
select floor(count * 0.9) as fin, r.amount > floor(count * 0.9) as fin_2 from temp_count as t
join rations as r
on r.amount != t.count
              
                
```

## 3 задание:

```sql
with temp as (
select *, extract('month' from date) as md, extract('year' from current_date) - extract('year' from date) as y1  from honeycomb_game
),
t1 as (
select * from temp
join monthly_temperatures as m2
on temp.md = m2.month
  where y1 <= 20
)

select shape, month, avg(average_completion_time) as fin1 from t1
where avg_temperature = (select max(avg_temperature) from t1) or
avg_temperature = (select min(avg_temperature) from t1)
group by shape, month
order by fin1

                
                
```



## 4 задание

```sql
with temp as (
select team_id, avg(age) as average_age,
case 
when avg(age) < 40 then 'Fit'
when avg(age) >= 40 and avg(age) <= 50 then 'Grizzled'
when avg(age) > 50 then 'Elderly'
end
as age_group
from player
group by team_id
  having count(*) = 10
)
select team_id, average_age, age_group,
rank() over(order by average_age desc)
from temp
```


## 5 задание

```sql
with t as (
select player1_id, player2_id, count(*) from Daily_interactions as d
join player as p
on d.player1_id = p.id or d.player2_id = p.id
where (player1_id = 456 or player2_id = 456) and (status = 'alive')
group by player1_id, player2_id
  ),
fin_1 as (
  select greatest(player1_id, player2_id) as gg, least(player1_id, player2_id) as fr, sum(count)
  from t
  group by greatest(player1_id, player2_id), least(player1_id, player2_id)
  order by sum desc
  limit 1
 ),
fin_2 as (
 select gg, fr, first_name as gg_name, sum from fin_1
 join player as p 
 on p.id = fin_1.gg
 ),
 fin_3 as (
 select gg_name, first_name, sum from fin_2
   join player as p
   on p.id = fin_2.fr
 )
 
 select * from fin_3
```

## 6 задание

```sql
with hard_game as (
select game_type, count(*) from suppliers as s
join equipment as e
on s.id = e.supplier_id
join failure_incidents as f
on f.failed_equipment_id = e.id
group by game_type
order by count desc
limit 1
),
hard_supplier as (
select name, count(*) from suppliers as s
join equipment as e
on s.id = e.supplier_id
join failure_incidents as f
on f.failed_equipment_id = e.id
where game_type = (select game_type from hard_game)
group by name
order by count desc
  limit 1
),
min_failure as (
select e.id, min(failure_date) as first_failure_date from suppliers as s
join equipment as e
on s.id = e.supplier_id
join failure_incidents as f
on f.failed_equipment_id = e.id
  where game_type = (select game_type from hard_game) 
  and name = (select name from hard_supplier)
  group by e.id
)
  
select 
       floor(avg((first_failure_date - e.installation_date) / 365.2425)) as avg_lifespan_years
from suppliers as s
join equipment as e on s.id = e.supplier_id
join min_failure as ff on e.id = ff.id
where game_type = (select game_type from hard_game)
  and name = (select name from hard_supplier)
group by name, game_type; 
            
                
```

## 7 задание

```sql

           with tempo as (
select g.id, g.code_name, g.status, r.last_check_time, c.location, c.movement_detected_time, (c.movement_detected_time - r.last_check_time) as lol
from guard as g
join room as r
on r.id = g.assigned_room_id
join camera as c
on c.guard_spotted_id = g.id
),
fin as (
select *,
max(movement_detected_time) over(order by id) as t1,
  min(movement_detected_time) over(rows between unbounded preceding and unbounded following) as t2
from tempo
  order by id
)
select id as guard_number,
code_name, 
status, 
last_check_time, 
location, 
movement_detected_time,
lol as outside_room, 
t1 - t2 as fin_time  from fin     
                
```


## 8 задание

```sql

with all_players as (
  select p.id, first_name, last_name, last_moved_time_seconds from player as p
join glass_bridge as g
on p.game_id = g.id
where death_description = 'pushed'
)
select * from all_players
order by last_moved_time_seconds desc
limit 1  
                
```

## 9 задание

```sql
WITH disappearance_window as (
    select date, start_time, end_time
    from game_schedule
    where type = 'Squid Game'
    order by date desc
    limit 1
),
guards_away as (
    select g.id as guard_id, g.assigned_post, g.shift_start, g.shift_end, dal.door_location, dal.access_time
    from guard g
    join disappearance_window dw
    on g.shift_start < dw.end_time and g.shift_end > dw.start_time
    left join daily_door_access_logs dal
    on dal.guard_id = g.id
    and dal.access_time between g.shift_start and g.shift_end
    where g.assigned_post != dal.door_location
),
suspicious_access as (
    select
        g.id as potential_associate_id,
        dal.access_time as time_accessed
    from daily_door_access_logs dal
    join guard g on g.id = dal.guard_id
    where dal.door_location = 'Upper Management'
    and dal.access_time between '11:00:00'::time and '12:00:00'::time
    and g.id != 31
)
select * from suspicious_access;
```