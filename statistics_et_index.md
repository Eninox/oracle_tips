
# Statistiques

## Outil dbms_stats
Package de collecte des statistiques de l'optimiseur pour une table, colonne, index, système, afin d'analyser les requêtes et choisir l'exécution la plus optimisée (coût de ressources le plus bas).
> Version 19c introduit l'automatisation de la collecte de statistiques de l'optimiseur avec DBMS_STATS

## Vérifier dernières analyses des stat
```sql
SELECT TABLE_NAME, LAST_ANALYZED, SAMPLE_SIZE, AVG_ROW_LEN, EMPTY_BLOCKS
FROM USER_TABLES;
```
## Lancer les stat sur une table (syntaxe 'schema','table')
```sql
EXEC dbms_stats.gather_table_stats('HR', 'EMPLOYEES');
```

## Suivi requêtes longues temps réel (ex pour stat)
```sql
select SID, opname , START_TIME,TOTALWORK, sofar, (sofar/totalwork) * 100 done,
sysdate + TIME_REMAINING/3600/24 end_at
from v$session_longops
where totalwork > sofar
order by 7;
```

# Index
Objet d'une table qui pointe sur des colonnes, afin d'accélérer l'itération des lignes interrogées et optimiser les ressources/temps de réponse.

Réindexation à faire régulièrement lorsqu'une table enregistre plusieurs écritures/suppressions.

## Affichage de toutes les tables de la vue dba_indexes
```sql
select table_name,count(index_name) from dba_indexes 
group by table_name
order by count(index_name);
```

## Lancer la réindexation pour une table
> Attention gourmand, le prévoir en HNO
```sql
ALTER INDEX table
REBUILD;
```