# 项目准备

### 创建工程

```bash
#创建项目
#django-admin startproject meiduo_mall
```


### 配置开发环境

```bash
#准备配置文件目录
#meiduo_mall/meiduo_mall/settings
#新建包，命名为settings，作为配置文件目录

#准备开发和生产环境配置文件
#在配置包settings中，新建开发和生产环境配置文件

#meiduo_mall/meiduo_mall/settings/dev.py
#准备开发环境配置内容
#将默认的配置文件settings.py中内容拷贝至dev.py
```


### 配置Jinja2模板引擎

```bash

#安装jinja2
#pip install Jinja2

#编写Jinja2模板引擎环境配置代码
#meiduo_mall/meiduo_mall/utils/jinja2_env.py
from jinja2 import Environment
from django.contrib.staticfiles.storage import staticfiles_storage
from django.urls import reverse


def jinja2_environment(**options):
    env = Environment(**options)
    env.globals.update({
        'static': staticfiles_storage.url,
        'url': reverse,
    })
    return env


"""
确保可以使用模板引擎中的{{ url('') }} {{ static('') }}这类语句 
"""


#配置Jinja2模板引擎  在原有TEMPLATES上添加
#meiduo_mall/meiduo_mall/settings/dev.py
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.jinja2.Jinja2',  # jinja2模板引擎
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
            # 补充Jinja2模板引擎环境
            'environment': 'meiduo_mall.utils.jinja2_env.jinja2_environment', 
        },
    },
]

#新建templates包
#meiduo_mall/meiduo_mall/templates
```
### 配置MySQL数据库

```bash
#配置MySQL数据库

#安装驱动程序
#MySQLdb只适用于Python2.x的版本，Python3.x的版本中使用PyMySQL替代MySQLdb
#pip install PyMySQL

#meiduo_mall/meiduo_mall/__init__.py
#在工程同名子目录的__init__.py文件中，添加如下代码：
import pymysql
pymysql.version_info = (1, 4, 13, "final", 0)
pymysql.install_as_MySQLdb()

#新建MySQL数据库
#1.新建MySQL数据库：meiduo_mall
#$ create database meiduo charset=utf8;

#2.新建MySQL用户
#$ create user itheima identified by '123456';

#3.授权itcast用户访问meiduo_mall数据库
#$ grant all on meiduo.* to 'itheima'@'%';

#4.授权结束后刷新特权
#$ flush privileges;
```
### 配置Redis数据库
```bash
#安装django-redis扩展包
#pip install django-redis
#meiduo_mall/meiduo_mall/settings/dev.py 添加以下内容
#配置Redis数据库
CACHES = {
    "default": { # 默认
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/0",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
    "session": { # session
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
        }
    },
}
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "session"
```
### 配置工程日志
```bash
#meiduo_mall/meiduo_mall/settings/dev.py 添加以下内容
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,  # 是否禁用已经存在的日志器
    'formatters': {  # 日志信息显示的格式
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(lineno)d %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(module)s %(lineno)d %(message)s'
        },
    },
    'filters': {  # 对日志进行过滤
        'require_debug_true': {  # django在debug模式下才输出日志
            '()': 'django.utils.log.RequireDebugTrue',
        },
    },
    'handlers': {  # 日志处理方法
        'console': {  # 向终端中输出日志
            'level': 'INFO',
            'filters': ['require_debug_true'],
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'file': {  # 向文件中输出日志
            'level': 'INFO',
            'class': 'logging.handlers.RotatingFileHandler',
            'filename': os.path.join(os.path.dirname(BASE_DIR), 'logs/meiduo.log'),  # 日志文件的位置
            'maxBytes': 300 * 1024 * 1024,
            'backupCount': 10,
            'formatter': 'verbose'
        },
    },
    'loggers': {  # 日志器
        'django': {  # 定义了一个名为django的日志器
            'handlers': ['console', 'file'],  # 可以同时向终端与文件中输出日志
            'propagate': True,  # 是否继续传递日志信息
            'level': 'INFO',  # 日志器接收的最低日志级别
        },
    }
}

#准备日志文件目录  新建文件夹
#meiduo_mall/logs

#meiduo_mall/logs中建立一个 .gitkeep 文件，然后即可提交。 用来保证logs目录被git记录
#代码提交到Git仓库时logs文件目录保留 meiduo_mall/logs/*.log会被忽略

```

# 配置前端静态文件

```bash
#STATIC_URL = '/static/'
#meiduo_mall/meiduo_mall/settings/dev.py 添加以下内容
# 配置静态文件加载路径
#STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
```

