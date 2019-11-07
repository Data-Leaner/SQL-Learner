## PostGIS
### 创建GIS数据库
```sql
-- 创建数据库
create database database_name;

-- 创建扩展
create extension postgis;

-- 查看gis的版本
select postgis_full_version();

-- 查看gis是否安装成功
select name, default_version
  from pg_avaliable_extensions
 where name ~ 'gis';
```

对于PostGIS而言，所有几何对象都有一个公共父类Geometry,这种面向对象的组织形式允许在数据库中进行一些灵活的操作：例如在数据表中的同一列中存储不同的几何对
象，每种几何对象实际上都是PostGIS底层几何库geos中对象的包装。

### 几何对象的输入
- 众所周知的文本格式(WKT, Well.Known.Text)
- 众所周知的二进制格式(WKB, Well-Known・Binary)
- GeoJSON等编码
- 返回几何类型的函数

### 



