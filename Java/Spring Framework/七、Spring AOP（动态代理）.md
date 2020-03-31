## 一、概述
1. 问题
> 1. 完善user案例
> 2. 分析案例中的问题
> 3. 动态代理技术
> 4. 动态代理的另一种实现方式
> 5. 解决案例中的问题
> 6. AOP概念
> 7. spring中AOP的相关术语
> 8. spring中基于XML和注解的AOP配置

## 一、问题

#### 1.1 业务场景
在转账的业务中，需要保证转出方扣钱成功，同时要保证转入方收到相同金额的钱，这就要求整个转账的过程在同一个事务中

1. 业务层代码
```
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    private TransactionManager txManager;

    // 使用set注入dao
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void setTxManager(TransactionManager txManager) {
        this.txManager = txManager;
    }

    @Override
    public int addOne(User user) {
        try {
            // 1. 开启事务
            txManager.start();
            // 2. 执行操作
            int count = userDao.addOne(user);
            // 3. 提交事务
            txManager.commit();
            // 4. 返回数据
            return count;
        } catch (Exception e) {
            // 5. 异常回滚
            txManager.rollBack();
        } finally {
            // 6. 结束事务
            txManager.release();
        }
        return 0;
    }

    @Override
    public int deleteOne(int id) {
        try {
            // 1. 开启事务
            txManager.start();
            // 2. 执行操作
            int count = userDao.deleteOne(id);
            // 3. 提交事务
            txManager.commit();
            // 4. 返回数据
            return count;
        } catch (Exception e) {
            // 5. 异常回滚
            txManager.rollBack();
        } finally {
            // 6. 结束事务
            txManager.release();
        }
        return 0;
    }

    @Override
    public int updateOne(User user) {
        try {
            // 1. 开启事务
            txManager.start();
            // 2. 执行操作
            int count = userDao.updateOne(user);
            // 3. 提交事务
            txManager.commit();
            // 4. 返回数据
            return count;
        } catch (Exception e) {
            // 5. 异常回滚
            txManager.rollBack();
        } finally {
            // 6. 结束事务
            txManager.release();
        }
        return 0;
    }

    @Override
    public User findOne(int id) {
        try {
            // 1. 开启事务
            txManager.start();
            // 2. 执行操作
            User user = userDao.findOne(id);
            // 3. 提交事务
            txManager.commit();
            // 4. 返回结果
            return user;
        } catch (Exception e) {
            // 5. 异常回滚
            txManager.rollBack();
        } finally {
            // 6. 结束事务
            txManager.release();
        }
        return null;
    }

    @Override
    public List<User> findAll() {
        try {
            // 1. 开启事务
            txManager.start();
            // 2. 执行操作
            List<User> list = userDao.findAll();
            // 3. 提交事务
            txManager.commit();
            // 4. 返回结果
            return list;
        } catch (Exception e) {
            // 5. 异常回滚
            txManager.rollBack();
        } finally {
            // 6. 结束事务
            txManager.release();
        }
        return null;
    }

    /**
     * 转账操作
     * @author: lizza@vizen.cn
     * @date: 2020/3/20 11:36 下午
     * @param originAccount
     * @param targetAccount
     * @param money
     * @return int
     */
    @Override
    public int transferMoney(Integer originAccount, Integer targetAccount, Double money) {
        try {
            // 1. 开启事务
            txManager.start();
            // 2. 执行操作
            // 2.1 查出转出账户
            User originUser = userDao.findOne(originAccount);
            // 2.2 查出站如账户
            User targetUser = userDao.findOne(targetAccount);
            // 2.3 转出账户减少金额
            double originMoney = originUser.getMoney() - money;
            originUser.setMoney(originMoney);
            // 2.4 转入账户增加金额
            double targetMoney = targetUser.getMoney() + money;
            targetUser.setMoney(targetMoney);
            // 2.5 更新转出账户
            userDao.updateOne(originUser);
            int num = 1/0;
            // 2.6 更新转入账户
            userDao.updateOne(targetUser);
            // 3. 提交事务
            txManager.commit();
            // 4. 返回结果
            return 1;
        } catch (Exception e) {
            // 5. 异常回滚
            txManager.rollBack();
            e.printStackTrace();
        } finally {
            // 6. 结束事务
            txManager.release();
        }
        return 0;
    }
}

```
> - 问：事务控制为什么需要加到业务层？
> - 答：通常一个业务需要连续多次操作数据库，如果中间某一次操作数据库的时候发生了异常，整个业务没有回滚就提交了部分操作，就会发生异常；比如最常见的转账业务，如果转出方转出成功，而转入的时候发生了异常，就会造成转入方没有收到转账，从而造成金额损失

2. beans.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 配置service -->
    <bean id="userService" class="com.lizza.service.impl.UserServiceImpl">
        <!-- 注入Dao -->
        <property name="userDao" ref="userDao"></property>
        <property name="txManager" ref="txManager"></property>
    </bean>

    <!-- 配置事务管理器 -->
    <bean name="txManager" class="com.lizza.util.TransactionManager">
        <property name="jdbcUtil" ref="jdbcUtil"></property>
    </bean>

    <!-- 配置dao -->
    <bean id="userDao" class="com.lizza.dao.impl.UserDaoImpl">
        <!-- 注入QueryRunner -->
        <property name="queryRunner" ref="queryRunner"></property>
        <property name="jdbcUtil" ref="jdbcUtil"></property>
    </bean>

    <!-- 配置JdbcUtil -->
    <bean id="jdbcUtil" class="com.lizza.util.JdbcUtil">
        <!-- 注入数据源 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置QueryRunner；QueryRunner需要配置成多例模式，避免发生线程安全问题 -->
    <bean id="queryRunner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype"></bean>

    <!-- 配置数据源 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3360/spring"/>
        <property name="user" value="root"/>
        <property name="password" value="root"/>
    </bean>
</beans>
```

#### 1.2 存在问题
1. 代码大量重复：每一个涉及到事务的控制都需要增加事务控制的代码
2. 代码重复导致耦合度增大，开发维护成本升高：新的人接手代码时，需要更多的时间去了解与业务无关的事务控制的代码

#### 1.3 解决方案
静态代理，动态代理（本案例选择动态代理）

## 二、动态代理

#### 2.1 概述
动态的在运行期创建某个类或者接口的代理类

#### 2.2 分类
1. JDK动态代理
2. cglib动态代理

类型|特点|使用方式
-|-|-
JDK动态代理|基于接口的动态代理|使用Proxy的newProxyInstance()方法创建代理对象
cglib动态代理|基于类的动态代理|使用Enhancer类中的create()方法创建代理对象

#### 2.3 JDK动态代理示例
```
public class Client {

    /**
     * JDK动态代理
     *  定义：可以在运行期动态创建某个interface的实例
     *  作用：在不修改源码的基础上实现对方法的增强
     *  特点：需要接口，并且实现类的实例
     *  分类：
     *      1. 基于接口的动态代理
     *      2. 基于子类的动态代理
     * 基于接口的动态代理
     *  涉及的类：Proxy
     *  提供者：JDK官方
     * 创建动态代理的要求
     *  被代理的类最少实现一个接口，如果没有则不能使用
     * newProxyInstance方法的参数
     *  ClassLoader：类加载器
     *      用于加载代理对象的字节码；与被代理对象使用相同的类加载器；固定写法
     *  Class<?>[]：字节码数组
     *
     *  InvocationHandler：用于提供增强的代码
     *      用于实现增强的代码；通常情况下是匿名内部类；非必需
     */
    public static void main(String[] args){
        Producer producer = new Producer();
        IProducer proxyProducer = (IProducer) Proxy.newProxyInstance(producer.getClass().getClassLoader(),
                producer.getClass().getInterfaces(),
                new InvocationHandler() {
                    /**
                     * 作用：执行被代理对象的任何接口方法时都会执行该方法
                     * @author: lizza@vizen.cn
                     * @date: 2020/3/23 9:19 下午
                     * @param proxy     代理对象的引用
                     * @param method    当前执行的方法
                     * @param args      当前执行方法所需的参数
                     * @return          和被代理对象具有相同的返回值
                     */
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        // 1. 获取传入的参数
                        if("sale".equals(method.getName())) {
                            return method.invoke(producer, (float) args[0] * 0.8f);
                        }
                        return method.invoke(producer, args);
                    }
                }
        );
        proxyProducer.sale(10000);
    }
}
```
#### 2.4 cglib动态代理示例
```
public class Client {

    /**
     * cglib动态代理
     *  定义：可以在运行期动态创建某个class的实例
     *  作用：在不修改源码的基础上实现对方法的增强
     *  特点：对指定类进行代理
     *  分类：
     *      1. 基于接口的动态代理
     *      2. 基于子类的动态代理
     * 基于接口的动态代理
     *  涉及的类：Proxy
     *  提供者：第三方cglib库
     * 创建动态代理的要求
     *  被代理的类不能用final修饰
     * 如何创建代理对象
     *  使用Enhancer类中的create方法
     * create方法的参数
     *  ClassLoader：类加载器
     *      用于加载代理对象的字节码；与被代理对象使用相同的类加载器；固定写法
     *  Class<?>[]：字节码数组
     *
     *  Callback：用于提供增强的代码
     *      用于实现增强的代码；通常情况下是匿名内部类；非必需
     *      一般创建MethodInterceptor
     */
    public static void main(String[] args){
        Producer producer = new Producer();
        Producer cglibProducer = (Producer) Enhancer.create(producer.getClass(), producer.getClass().getInterfaces(), new MethodInterceptor() {
            /**
             * 执行被代理对象的任何方法都会经过该方法
             * @author: lizza@vizen.cn
             * @date: 2020/3/25 8:21 下午
             * @param proxy
             * @param method
             * @param args
             * @param methodProxy
             * @return java.lang.Object
             */
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                // 1. 获取传入的参数
                if("sale".equals(method.getName())) {
                    return method.invoke(producer, (float) args[0] * 0.8f);
                }
                return method.invoke(producer, args);
            }
        });
        cglibProducer.sale(12000);
    }
}
```

## 三、用动态代理解决问题
#### 3.1 代码示例
1. 创建动态代理对象并增加事务控制
```
public class BeanFactory {

    private UserService userService;

    private TransactionManager txManager;

    public UserService getUserService() {
        return (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(),
                userService.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        Object result = null;
                        try {
                            txManager.start();
                            result = method.invoke(userService, args);
                            txManager.commit();
                        } catch (Exception e) {
                            txManager.rollBack();
                            e.printStackTrace();
                        } finally {
                            txManager.release();
                        }
                        return result;
                    }
                });
    }

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public void setTxManager(TransactionManager txManager) {
        this.txManager = txManager;
    }
}
```
2. 配置beans.xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 配置带事务控制的service -->
    <bean id="beanFactory" class="com.lizza.factory.BeanFactory">
        <property name="txManager" ref="txManager"></property>
        <property name="userService" ref="userService"></property>
    </bean>
    <bean id="proxyUserService" factory-bean="beanFactory" factory-method="getUserService"></bean>

    <!-- 配置service -->
    <bean id="userService" class="com.lizza.service.impl.UserServiceImpl">
        <!-- 注入Dao -->
        <property name="userDao" ref="userDao"></property>
    </bean>

    <!-- 配置事务管理器 -->
    <bean name="txManager" class="com.lizza.util.TransactionManager">
        <property name="jdbcUtil" ref="jdbcUtil"></property>
    </bean>

    <!-- 配置dao -->
    <bean id="userDao" class="com.lizza.dao.impl.UserDaoImpl">
        <!-- 注入QueryRunner -->
        <property name="queryRunner" ref="queryRunner"></property>
        <property name="jdbcUtil" ref="jdbcUtil"></property>
    </bean>

    <!-- 配置JdbcUtil -->
    <bean id="jdbcUtil" class="com.lizza.util.JdbcUtil">
        <!-- 注入数据源 -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置QueryRunner；QueryRunner需要配置成多例模式，避免发生线程安全问题 -->
    <bean id="queryRunner" class="org.apache.commons.dbutils.QueryRunner" scope="prototype"></bean>

    <!-- 配置数据源 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/spring"/>
        <property name="user" value="root"/>
        <property name="password" value="root"/>
    </bean>
</beans>
```
 3. 进行测试
```
public class UserServiceTest {

    @Test
    public void addOne() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserService userService = context.getBean("userService", UserService.class);

        User user = new User();
        user.setId(4);
        user.setName("test");
        user.setAge(19);

        Assert.assertEquals(1, userService.addOne(user));
    }

    @Test
    public void deleteOne() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserService userService = context.getBean("userService", UserService.class);

        Assert.assertEquals(1, userService.deleteOne(4));

    }

    @Test
    public void updateOne() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserService userService = context.getBean("userService", UserService.class);
        User user = new User();
        user.setId(4);
        user.setName("test");
        user.setAge(0);
        Assert.assertEquals(0, userService.updateOne(user));
    }

    @Test
    public void findOne() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserService userService = context.getBean("userService", UserService.class);
        User user = userService.findOne(1);
        Assert.assertEquals(1, user.getId());
    }

    @Test
    public void findAll() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserService userService = context.getBean("userService", UserService.class);
        List<User> list = userService.findAll();
        System.out.println(list);
        Assert.assertNotNull(list);
    }

    @Test
    public void transferMoney() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        // 获取代理对象
        UserService userService = context.getBean("proxyUserService", UserService.class);
        userService.transferMoney(1, 2, 100.0);
        List<User> list = userService.findAll();
        System.out.println(list);
    }
}
```
#### 3.2 总结
1. 可以看到动态代理在不改变源码的基础上实现了事务的控制
2. 事务的控制类独立于业务代码，降低了耦合性

## 四、Spring AOP
#### 4.1 AOP概述
1. 定义
> AOP（Aspect Oriented Programming）面向切面编程，通过预编译和运行期动态代理的方式，实现了程序各层级业务逻辑的隔离，降低了程序的耦合性，提高了程序开发的效率

2. 作用
> 在程序运行期间，通过动态代理的方式不改变源码实现对方法的增强

3. 优势
> 1. 降低了代码的耦合性
> 2. 提高了开发效率
> 3. 方便维护

> 源码地址：[https://github.com/KJGManGlory/spring-framework](https://github.com/KJGManGlory/spring-framework)
