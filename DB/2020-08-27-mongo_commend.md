---
layout: post
title: mongoDB Commend
date: 2020-08-27 09:20:23 +0900
category: mongoDB
---

# Mongo DB 명령어

> RDMS의 명령어는 익숙하지만 NoSql인 MongoDB의 명령어가 익숙하지 않아 정리할 겸 oracle 쿼리문과 비교하여 비슷한 기능을 하는 명령어를 비교하여 정리해두려고 한다.

### CREATE

> ##### 데이터베이스 생성

```
use my_mongo
```

> ##### user 테이블 생성

```
db.createCollection("user")
```

### SELECT

> ##### SELECT \* FROM USER

```
db.user.find()
```

> ##### query plan 보기

```
db.user.find().explain()
```

> ##### SELECT USER_ID, STATUS, AGE FROM USER WHERE STATUS **=** 'A'

```
db.user.find({status:"A"},{user_id:1,status:1,age:1,_id:0})
```

> ##### SELECT \* FROM USER WHERE STATUS **!=** 'abc001'

```
db.user.find({status:{$ne:"abc001"}})
```

> ##### SELECT \* FROM USER WHERE AGE > 25 **and** age <=50

```
db.user.find({age:{$gt:25,$lte:50}})
```

> ##### SELECT \* FROM USER WHERE STATUS = 'B' **or** age = 25

```
db.user.find({$or:[{staus:"B"},{age:25}]})
```

> ##### SELECT \* FROM USER WHERE USER_ID LIKE '%cd%'

```
db.user.find({user_id:{$regex:/cd/}})
```

> ##### SELECT \* FROM USER WHERE USER_ID LIKE 'bc%'

```
db.user.find({user_id:{$regex:/^bc/}})
```

> ##### SELECT \* FROM USER WHERE USER_ID LIKE '%01'

```
db.user.find({user_id:{$regex:/01$/}})
```

> ##### SELECT USER_ID, AGE, STATUS FROM USER WHERE STATUS IN ('A', 'B') AND AGE >=25 AND AGE <=45

```
db.user.find({status:{$in:['A','B']},age:{$gte:25,$lte:45}},{user_id:1, age:1,status:1}).sort({user_id:-1})
```

> ##### SELECT \* FROM USER WHERE AGE > 20 limit 1

```
db.user.findOne({age:{$gt:20}})
```

> ##### SELECT COUNT(\*) FROM USER WHERE AGE > 30

```
db.user.count({age:{$gt:30}})
```

> ##### SELECT COUNT(USER_ID) FROM USER

```
db.user.count({user_id:{$exists:true}})
```

> ##### SELECT DISTINCT(STATUS) FROM USER

```
db.user.aggregate([{$group:{_id:"$status"}}])
```

### INSERT

> ##### INSERT INTO USER (user_id, age, status)

```
db.user.insertMany([
{ user_id: "bcd001", age:25, status:"A" },
 { user_id: "bcd002", age:65, status:"A" },
 { user_id: "bcd003", age:35, status:"A" },
 { user_id: "bcd004", age:27, status:"B" },
 { user_id: "bcd005", age:60, status:"C" },
 { user_id: "bcd006", age:15, status:"C" },
 { age:14, status:"C" } ])
```

### UPDATE

> ##### UPDATE USER SET STATUS = 'C' WHERE AGE > 45

```
db.user.updateMany({age:{$gt:45}},{$set:{status:'C'}})
```

> ##### UPDATE USER SET STATUS = 'D' WHERE AGE <= 25

```
db.user.updateOne({age:{$lte:25}},{$set:{status:'D'}})
```

> ##### UPDATE USER SET AGE = AGE+3 WHERE STATUS = 'A'

```
db.user.updateMany({status:'A'},{$inc:{age:3}})
```

### DELETE

> ##### DELETE FROM USER WHERE STATUS = 'D'

```
db.user.deleteMany({status:'D'})
```

> ##### name 필드가 있는 document를 삭제

```
db.user.deleteMany({name:{$exists:true}})
```
