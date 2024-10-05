
# Stockage

## Taille base
Se baser sur le HWM (High Water Mark -> niveau des hautes eaux)
		
> Se situe à l'endroit de la dernière écriture dans la base
même si des lignes de la table sont supprimées, l'espace alloué reste le même puisqu'il est réservé dans l'extent
sur 1 bloc il y a 8 compartiments avec X lignes, présentes à un instant mais peuvent être supprimées, laissant l'espace vide

Possède les dernières infos d'écriture stockage et contient les infos tbs, cpu, …

```sql
SELECT HIGHWATER/1024/1024, LAST_VALUE/1024/1024
FROM dba_high_water_mark_statistics
where name like 'DB_SIZE';
```

> Attention ajouter un /1024 en Gibi

