Postgresql
======

# install

# 扩展

### PostGIS

PostGIS is a spatial database extender for [PostgreSQL](https://postgresql.org/) object-relational database. It adds support for geographic objects allowing location queries to be run in SQL.

```sql
SELECT superhero.name
FROM city, superhero
WHERE ST_Contains(city.geom, superhero.geom)
AND city.name = 'Gotham';
```

[Docs for latest stable release](http://postgis.net/docs/)

