# 用户注册

### 1. 展示用户注册页面

- #### 	创建用户模块子应用

  ```shell
  #创建apps包  子应用放在这个目录下
  #meiduo_mall/meiduo_mall/apps
  
  #在apps包下创建应用users
  # cd ~/projects/meiduo_project/meiduo_mall/meiduo_mall/apps
  # python ../../manage.py startapp users
  
  ```

- #### 	追加导包路径

  ```shell
  # 查看导包路径
  # meiduo_mall/meiduo_mall/settings/dev.py 添加以下内容
  # import sys
  # print(sys.path)
  
  # 'meiduo_project/meiduo_mall/meiduo_mall/apps'
  #查看项目BASE_DIR
  #'~/projects/meiduo_project/meiduo_mall/meiduo_mall'
  #追加导包路径
  #sys.path.insert(0, os.path.join(BASE_DIR, 'apps'))
  # print(sys.path)
  
  #注册用户模块应用
  #在dev.py的INSTALLED_APPS列表中添加，'users', # 用户模块应用
  
  INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
  
      'users', # 用户模块应用
  ]
  
  ```
- #### 	展示用户注册页面
  ```shell
  
  #加载页面静态文件 meiduo_mall/meiduo_mall/templates/register.html
  #修改
  <head>
      <meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
      <title>美多商城-注册</title>
      <link rel="stylesheet" type="text/css" href="{{ static('css/reset.css') }}">
      <link rel="stylesheet" type="text/css" href="{{ static('css/main.css') }}">
  </head>
  
  #定义用户注册视图
  #meiduo_mall/meiduo_mall/apps/users/views.py
  class RegisterView(View):
      """用户注册"""
  
      def get(self, request):
          """
          提供注册界面
          :param request: 请求对象
          :return: 注册界面
          """
          return render(request, 'register.html')
          
  #定义用户注册路由
  	#1.总路由
      urlpatterns = [
          # users
          url(r'^', include('users.urls', namespace='users')),
      ]
      
  	#2.子路由
  	app_name = 'users'#3.0django 需要添加app_name
      urlpatterns = [
          # 注册
          url(r'^register/$', views.RegisterView.as_view(), name='register'),
      ]
  ```

### 2. 用户模型类

- #### 定义用户模型类
 ```shell
#Django默认用户认证系统
    #Django自带用户认证系统
        #它处理用户账号、组、权限以及基于cookie的用户会话。
    #Django认证系统位置
        #django.contrib.auth包含认证框架的核心和默认的模型。
        #django.contrib.contenttypes是Django内容类型系统，它允许权限与你创建的模型关联。
    #Django认证系统同时处理认证和授权
        #认证：验证一个用户是否它声称的那个人，可用于账号登录。
        #授权：授权决定一个通过了认证的用户被允许做什么。
    #Django认证系统包含的内容
        #用户：用户模型类、用户认证。
        #权限：标识一个用户是否可以做一个特定的任务，MIS系统常用到。
        #组：对多个具有相同权限的用户进行统一管理，MIS系统常用到。
        #密码：一个可配置的密码哈希系统，设置密码、密码校验。
        
    #Django默认用户模型类
    #Django认证系统中提供了用户模型类User保存用户的数据。
        #User对象是认证系统的核心。
    #Django认证系统用户模型类位置
        #django.contrib.auth.models.User
 
	#父类AbstractUser介绍

    #User对象基本属性
        #创建用户(注册用户)必选： username、password
        #创建用户(注册用户)可选：email、first_name、last_name、last_login、date_joined、					is_active 、is_staff、is_superuse
        #判断用户是否通过认证(是否登录)：is_authenticated
    #创建用户(注册用户)的方法
        #user = User.objects.create_user(username, email, password, **extra_fields)
   	#用户认证(用户登录)的方法
        #from django.contrib.auth import authenticate
        #user = authenticate(username=username, password=password, **kwargs)
    #处理密码的方法
        #设置密码：set_password(raw_password)
        #校验密码：check_password(raw_password)
        
    # 自定义用户模型类 
    #meiduo_mall/meiduo_mall/apps/users/models.py
    from django.db import models
    from django.contrib.auth.models import AbstractUser

    # Create your models here.
	class User(AbstractUser):
        """自定义用户模型类"""
        mobile = models.CharField(max_length=11, unique=True, verbose_name='手机号')

        class Meta:
            db_table = 'tb_users'
            verbose_name = '用户'
            verbose_name_plural = verbose_name

        def __str__(self):
            return self.username

 ```

#### 迁移用户模型类

 ```shell
#思考：为什么Django默认用户模型类是User？
	#阅读源代码：'django.conf.global_settings'
        #AUTH_USER_MODEL = 'auth.User'
    #结论：
        #Django用户模型类是通过全局配置项 AUTH_USER_MODEL 决定的
    #配置规则：
    #AUTH_USER_MODEL = '应用名.模型类名'
    
    
# 指定本项目用户模型类 #meiduo_mall/meiduo_mall/settings/dev.py 添加以下内容
AUTH_USER_MODEL = 'users.User'

#创建迁移文件
#python manage.py makemigrations
#执行迁移文件
#python manage.py migrate


 ```

### 3.用户注册业务实现

- #### 保存注册数据

```shell
# 保存注册数据
try:
    User.objects.create_user(username=username, password=password, mobile=mobile)
except DatabaseError:
    return render(request, 'register.html', {'register_errmsg': '注册失败'})

# 响应注册结果
return http.HttpResponse('注册成功，重定向到首页')
```

- #### 状态保持

```shell
#login()方法介绍
   #用户登入本质：
    	#状态保持
    	#将通过认证的用户的唯一标识信息（比如：用户ID）写入到当前浏览器的 cookie 和服务端的 session 中。
    #login()方法：
    	#Django用户认证系统提供了login()方法。
    	#封装了写入session的操作，帮助我们快速登入一个用户，并实现状态保持。
    #login()位置：
    	#django.contrib.auth.__init__.py文件中。
		#login(request, user, backend=None)
		
#状态保持 session 数据存储的位置：Redis数据库的1号库
	SESSION_ENGINE = "django.contrib.sessions.backends.cache"
	SESSION_CACHE_ALIAS = "session"
```

- #### login()方法登入用户

```shell
# 保存注册数据
try:
    user = User.objects.create_user(username=username, password=password, mobile=mobile)
except DatabaseError:
    return render(request, 'register.html', {'register_errmsg': '注册失败'})

# 登入用户，实现状态保持
login(request, user)

# 响应注册结果
return redirect(reverse('contents:index'))
```

