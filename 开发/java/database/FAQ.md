### replace into vs insert into on duplicate key update

都是先删除原记录再插入，但是`insert into on duplicate key update`会在原记录的基础上进行修改，如果update后提交的数据不全还会使用原有的值，主键的值也不会自增。

### 大批量数据查询更新

使用临时表加快速度

