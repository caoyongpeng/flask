# SQLAlchemy操作 {#sqlalchemy操作}

## 1 新增 {#1-新增}

```
user = User(mobile='15612345678', name='itcast')
db.session.add(user)
db.session.commit()
profile = Profile(id=user.id)
db.session.add(profile)
db.session.commit()
```

对于批量添加也可使用如下语法

```
db.session.add_all([user1, user2, user3])
db.session.commit()
```

## 2 删除 {#4-删除}

* 方式一

  ```
    user = User.query.order_by(User.id.desc()).first()
    db.session.delete(user)
    db.session.commit()
  ```

* 方式二

  ```
   User.query.filter(User.mobile='18512345678').delete()
   db.session.commit()
  ```

## 3 更新 {#3-更新}

* 方式一

  ```
   user = User.query.get(1)
    user.name = 'Python'
    db.session.add(user)
    db.session.commit()
  ```

* 方式二

  ```
    User.query.filter_by(id=1).update({'name':'python'})
    db.session.commit()
  ```

##  {#4-删除}

## 4 事务 {#5-事务}

```
environ = {'wsgi.version':(1,0), 'wsgi.input': '', 'REQUEST_METHOD': 'GET', 'PATH_INFO': '/', 'SERVER_NAME': 'itcast server', 'wsgi.url_scheme': 'http', 'SERVER_PORT': '80'}

with app.request_context(environ):
    try:
        user = User(mobile='18911111111', name='itheima')
        db.session.add(user)
        db.session.flush() # 将db.session记录的sql传到数据库中执行
        profile = UserProfile(id=user.id)
        db.session.add(profile)
        db.session.commit()
    except:
        db.session.rollback()
```

## 5 查询 {#2-查询}

#### all\(\) {#all}

查询所有，返回列表

```
User.query.all()
```

#### first\(\) {#first}

查询第一个，返回对象

```
User.query.first()
```

#### get\(\) {#get}

根据主键ID获取对象，若主键不存在返回None

```
User.query.get(2)
```

#### 另一种查询方式 {#另一种查询方式}

```
db.session.query(User).all()
db.session.query(User).first()
db.session.query(User).get(2)
```

#### filter\_by {#filterby}

进行过虑

```
User.query.filter_by(mobile='13911111111').first()
User.query.filter_by(mobile='13911111111', id=1).first()  # and关系
```

#### filter {#filter}

进行过虑

```
User.query.filter(User.mobile=='13911111111').first()
```

#### 逻辑或 {#逻辑或}

```
from sqlalchemy import or_
User.query.filter(or_(User.mobile=='13911111111', User.name.endswith('号'))).all()
```

#### 逻辑与 {#逻辑与}

```
from sqlalchemy import and_
User.query.filter(and_(User.name != '13911111111', User.mobile.startswith('185'))).all()
```

#### 逻辑非 {#逻辑非}

```
from sqlalchemy import not_
User.query.filter(not_(User.mobile == '13911111111')).all()
```

#### offset {#offset}

偏移，起始位置

```
User.query.offset(2).all()
```

#### limit {#limit}

获取限制数据

```
User.query.limit(3).all()
```

#### order\_by {#orderby}

排序

```
User.query.order_by(User.id).all()  
# 正序

User.query.order_by(User.id.desc()).all()  
# 倒序
```

#### 复合查询 {#复合查询}

```
User.query.filter(User.name.startswith('13')).order_by(User.id.desc()).offset(2).limit(5).all()
```

```
query = User.query.filter(User.name.startswith('13'))
query = query.order_by(User.id.desc())
query = query.offset(2).limit(5)
ret = query.all()
```

#### 优化查询 {#优化查询}

```
user = User.query.filter_by(id=1).first()  # 查询所有字段
select user_id, mobile......

select * from   # 程序不要使用
select user_id, mobile,.... # 查询指定字段

from sqlalchemy.orm import load_only
User.query.options(load_only(User.name, User.mobile)).filter_by(id=1).first() # 查询特定字段
```

#### 聚合查询 {#聚合查询}

```
from sqlalchemy import func

db.session.query(Relation.user_id, func.count(Relation.target_user_id)).filter(Relation.relation == Relation.RELATION.FOLLOW).group_by(Relation.user_id).all()
```

#### 关联查询 {#关联查询}

##### 1. 使用ForeignKey {#1-使用foreignkey}

```
class User(db.Model):
    ...
    profile = db.relationship('UserProfile', uselist=False)
    followings = db.relationship('Relation')

class UserProfile(db.Model):
    id = db.Column('user_id', db.Integer, db.ForeignKey('user_basic.user_id'), primary_key=True,  doc='用户ID')
    ...

class Relation(db.Model):
    user_id = db.Column(db.Integer, db.ForeignKey('user_basic.user_id'), doc='用户ID')
    ...

# 测试   
user = User.query.get(1)
user.profile.gender
user.followings
```

##### 2. 使用primaryjoin {#2-使用primaryjoin}

```
class User(db.Model):
    ...

    profile = db.relationship('UserProfile', primaryjoin='User.id==foreign(UserProfile.id)', uselist=False)
    followings = db.relationship('Relation', primaryjoin='User.id==foreign(Relation.user_id)')

# 测试
user = User.query.get(1)
user.profile.gender
user.followings
```

##### 3. 指定字段关联查询 {#3-指定字段关联查询}

```
class Relation(db.Model):
    ...
    target_user = db.relationship('User', primaryjoin='Relation.target_user_id==foreign(User.id)', uselist=False)

from sqlalchemy.orm import load_only, contains_eager

Relation.query.join(Relation.target_user).options(load_only(Relation.target_user_id), contains_eager(Relation.target_user).load_only(User.name)).all()
```

##  {#3-更新}



