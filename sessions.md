
# Sessions

## Vue dictionnaire v$session

Consulter les operating system ayant une session lancée sur la base
```sql
select osuser, count(*) 
from v$session 
group by osuser 
order by count(*) 
desc;
```

Consulter les sessions actives (transaction temps réel sur la base)
```sql
select osuser, count(*) 
from v$session 
where status='ACTIVE' 
group by osuser 
order by count(*) 
desc;
```