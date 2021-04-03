## 2-IOC理论推导

控制反转是一种通过描述（XML或注解）并通过第三方去生产或获取特定对象的方式。在`spring`中实现控制反转的时`IoC`容器，其实现方法时依赖注入（`Dependency Injection,DI`）

由用户去控制所需的接口类型，程序员不需要设置接口类型，只需要暴露给用户即可


### 2.1-基础代码

#### 2.1.1-文件目录结构

- UserDao接口

- UserDaoImpl，Dao接口实现类
- UserService业务接口

- UserServiceImpl，service业务实现类

![image-20210327103237184](.\img\2-IOC理论推导\image-20210327103237184.png)

- iocstudy
  - dao
    - impl
      - UserDaoImpl(class)
      - UserDaoMysqlImpl(class)
      - UserDaoPracleImpl(class)
    - UserDao(Interface)
  - service
    - impl
      - UserServiceImpl(class)
    - UserService(Interface)

#### 2.1.2-文件代码

1. 首先，我们先写一个UserDao

   ```java
   package iocstudy.dao;
   
   public interface UserDao {
     void getUser();
   }
   ```

2. 其次，编写三个实现类

   ```java
   //UserDaoImpl
   package iocstudy.dao.impl;
   
   import iocstudy.dao.UserDao;
   
   public class UserDaoImpl implements UserDao {
     @Override
     public void getUser(){
       System.out.println("默认获取用户的数据");
     }
   }
   ```

   ```java
   //UserDaoMysqlImpl
   package iocstudy.dao.impl;
   
   import iocstudy.dao.UserDao;
   
   public class UserDaoMysqlImpl implements UserDao {
     @Override
     public void getUser(){
       System.out.println("Mysql获取用户数据");
     }
   }
   ```

   ```java
   //UserDaoOracleImpl
   package iocstudy.dao.impl;
   
   import iocstudy.dao.UserDao;
   
   public class UserDaoOracleImpl implements UserDao {
     @Override
     public void getUser(){
       System.out.println("Oracle获取用户数据");
     }
   }
   ```

3. 编写service层

   ```java
   //UserService
   package iocstudy.service;
   
   import iocstudy.dao.UserDao;
   
   public interface UserService {
     void getUser();
   }
   
   ```

   ```java
   //UserServiceImpl
   package iocstudy.service.impl;
   
   
   import iocstudy.dao.UserDao;
   import iocstudy.dao.impl.UserDaoImpl;
   import iocstudy.service.UserService;
   
   public class UserServiceImpl implements UserService {
     private UserDao userDao = new UserDaoImpl();
   
     @Override
     public void getUser(){
       userDao.getUser();
     }
   
   }
   
   ```

4. 编写测试类

   ```java
   import iocstudy.dao.impl.UserDaoMysqlImpl;
   import iocstudy.service.UserService;
   import iocstudy.service.impl.UserServiceImpl;
   
   public class MyTest {
     public static void main(String[] args){
       // 用户实际调用的是业务层，dao层他们不需要接触
       UserService userService = new UserServiceImpl();
   
       userService.getUser();
     }
   }
   
   ```

在上述业务中，每当用户的需求发生改变（所需的`DaoImpl`种类发生改变），程序员需进入`Service`层，修改`Impl`代码，本文中，需要修改的地方只有一处，还算方便，但如果程序代码量十分大，会导致修改代码的成本十分高，并且这种修改破坏了程序的完整性。

由此，引入`IOC（控制反转）`的思想，由用户去控制所需的接口类型，程序员不需要设置接口类型，只需要暴露给用户即可。

我们使用一个set接口实现，在`UserService`中创建`serUserDao`的方法，并在`UserServiceImpl`中实现

```java
//UserService
package iocstudy.service;

import iocstudy.dao.UserDao;

public interface UserService {
  void getUser();
  void setUserDao(UserDao userDao);
}
```

```java
//UserServiceImpl
package iocstudy.service.impl;

import iocstudy.dao.UserDao;
import iocstudy.service.UserService;

public class UserServiceImpl implements UserService {
  private UserDao userDao;

  @Override
  public void getUser(){
    userDao.getUser();
  }

  @Override
  public void setUserDao(UserDao userDao){
    this.userDao = userDao;
  }
}
```

- 之前，程序主动创建对象，控制权在程序员手上
- 使用了set注入后，程序不再具有主动性，而是变成了被动的接受对象

从本质上解决了问题，程序员不用再管理对象的创建，系统的耦合性大大降低