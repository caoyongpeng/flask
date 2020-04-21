### 配置方法 {#6--配置方法}

#### 方法1 {#方法1}

```
from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.executors.pool import ThreadPoolExecutor

executors = {
    'default': ThreadPoolExecutor(20),
}
scheduler = BackgroundScheduler(executors=executors)
```

#### 方法2 {#方法2}

```
from pytz import utc

from apscheduler.schedulers.background import BackgroundScheduler
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.executors.pool import ProcessPoolExecutor

executors = {
    'default': {'type': 'threadpool', 'max_workers': 20},
    'processpool': ProcessPoolExecutor(max_workers=5)
}

scheduler = BackgroundScheduler()

# .. 此处可以编写其他代码

# 使用configure方法进行配置
scheduler.configure(executors=executors)
```



