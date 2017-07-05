# MongoDB的命令行

- 安装mongoDB，并启动MongoDB服务
  基本流程就是： 下载 - 安装 - 启动服务
- 打开mongo命令行
```
cd F:\mongodb\bin
mongo
```
- 查询当前mongoDB服务有哪些数据库 
`show dbs`
- 使用某个数据库 
`use myTest`
- 查看当前使用的数据库下有哪些collection(集合) 
`show collections`
- 查看当前数据库，students集合中的记录数 
`db.students.count()`
- 查看当前数据库，students集合下的所有记录 
`db.students.find()`
- 查看当前数据库，students集合下匹配某些条件的所有记录 
`db.students.find({name: 'zhangsan'})`
- 查看当前数据库，students集合下匹配某些条件的第一条记录 
`db.students.findOne({name: 'zhangsan'})`
- 在当前数据库，students集合下增加一条记录 
`db.students.insert({name: 'lisi'})`
- 在当前数据库,students集合下，更新lisi的age字段 
`db.students.update({name: 'lisi'}, {age: 10})`
- 在当前数据库,students集合下，只更新lisi的age字段 
`db.students.update({name: 'lisi'}, {$set: {age: 10}})`
- 在当前数据库, students集合下删除一条记录 
`db.students.remove({name: 'lisi'})`
