# IOC XML配置

# IOC XML配置

# Spring_IOC

## 1. IOC

控制反转
将所有的组件(对象)交给IOC容器(工厂)进行控制，获取对象时向容器索取

### bean标签

属性

- id\name:给唯一标识(id能响应xsi的规范)
- class:指定类的全限定类名
- scope:指定对象的作用范围
  - singleton:默认值,单例
  - prototype:多例的
  - request:WEB项目中，Spring创建一个Bean对象，将对象存入request域中
  - session:WEB项目中，Spring创建一个Bean对象，将对象存入session域中
- init-method:指定类中的初始化方法名称
- destroy-method:指定类中销毁方法名称

bean的作用范围和生命周期

- 单例对象 scope=”singleton”

  一个应用只有一个对象的实例，作用范围为整个应用

  - 对象出生：当应用加载，创建容器时，对象就被创建了
  - 对象存货：只要容器在，对象一直存活
  - 对象死亡：当应用卸载，容器销毁时，对象被销毁
- 多例对象 scope=”prototype”

  每次访问对象，都会重新创建对象实例

  - 对象出生：当使用对象时，创建新的对象
  - 对象存活：只要对象在使用中，就一直存活
  - 对象死亡：当对象长时间不用时，被GC垃圾回收器回收

实例化Bean的三种方式

1. 使用默认的无参构造函数

   ```xml
   <bean id="accountService" class="com.zzxx.service.impl.AccountServiceImpl"/>
   ```
2. Spring管理静态工厂——使用静态工厂的方法创建对象

   ```java
   /**
   * 模拟⼀个静态⼯⼚，创建业务层实现类
   */
   public class StaticFactory {
      public static IAccountService createAccountService() {
   		 return new AccountServiceImpl();
   	 }
   }
   ```

   ```xml
   <!-- 此种⽅式是:
    使⽤ StaticFactory 类中的静态⽅法 createAccountService 创建对象，并存⼊ spring 容
   器
    id 属性: 指定 bean 的 id，⽤于从容器中获取
    class 属性: 指定静态⼯⼚的全限定类名
    factory-method 属性: 指定⽣产对象的静态⽅法
   -->
   <bean id="accountService" class="com.zzxx.factory.StaticFactory" factorymethod="createAccountService"/>
   ```
3. Spring管理实例工厂

   ```java
   /**
   * 模拟⼀个实例⼯⼚，创建业务层实现类
   * 此⼯⼚创建对象，必须现有⼯⼚实例对象，再调⽤⽅法
   */
   public class InstanceFactory {
   	public IAccountService createAccountService() {
   		return new AccountServiceImpl();
   	}
   }
   ```

   ```xml
   <!-- 此种⽅式是:
    先把⼯⼚的创建交给 spring 来管理。
    然后在使⽤⼯⼚的 bean 来调⽤⾥⾯的⽅法
    factory-bean 属性: ⽤于指定实例⼯⼚ bean 的 id。
    factory-method 属性: ⽤于指定实例⼯⼚中创建对象的⽅法。
   -->
   <bean id="instancFactory" class="com.zzxx.factory.InstanceFactory"/>
   <bean id="accountService" factory-bean="instancFactory" factorymethod="createAccountService"/>
   ```

## 2.依赖注入

依赖注入是Spring框架IOC的具体实现，直观的是把持久层对象传入业务层

1. 构造函数注入

   ```java
   public class AccountServiceImpl implements IAccountService {
   	private String name;
   	private Integer age;
   	private Date birthday;
   	public AccountServiceImpl(String name, Integer age, Date birthday) {
   		this.name = name;
   		this.age = age;
   		this.birthday = birthday;
   	}
   	@Override
   	public void saveAccount() {
   		System.out.println(name + "," + age + "," + birthday);
   	}
   }
   ```

   ```xml
   <!-- 使⽤构造函数的⽅式，给 service 中的属性传值
    要求:
    类中需要提供⼀个对应参数列表的构造函数。
    涉及的标签:constructor-arg
    属性:index: 指定参数在构造函数参数列表的索引位置
   			type: 指定参数在构造函数中的数据类型
   			name: 指定参数在构造函数中的名称 ⽤这个找给谁赋值
   	===上⾯三个都是找给谁赋值，下⾯两个指的是赋什么值的===
   			value: 它能赋的值是基本数据类型和 String 类型
   			ref: 它能赋的值是其他 bean 类型，也就是说，必须得是在配置⽂件中配置过的 bean
   -->
   <bean id="accountService" class="com.zzxx.service.impl.AccountServiceImpl">
   <constructor-arg name="name" value="张三"/>
   <constructor-arg name="age" value="18"/>
   <constructor-arg name="birthday" ref="now"/>
   </bean>
   <bean id="now" class="java.util.Date"/>
   ```
2. set方法注入

```java
public class AccountServiceImpl implements IAccountService {
	private String name;
	private Integer age;
	private Date birthday;
	public void setName(String name) {
		this.name = name;
	}
	public void setAge(Integer age) {
		this.age = age;
	}
	public void setBirthday(Date birthday) {
		this.birthday = birthday;
	}
	@Override
	public void saveAccount() {
		System.out.println(name + "," + age + "," + birthday);
	}
}
```

```xml
<!-- 通过配置⽂件给 bean 中的属性传值: 使⽤ set ⽅法的⽅式
涉及的标签:property
属性:
name: 找的是类中 set ⽅法后⾯的部分
ref: 给属性赋值是其他 bean 类型的
value: 给属性赋值是基本数据类型和 string 类型的实际开发中，此种⽅式⽤的较多。
-->
<bean id="accountService" class="com.zzxx.service.impl.AccountServiceImpl">
<property name="name" value="test"/>
<property name="age" value="21"/>
<property name="birthday" ref="now"/>
</bean>
<bean id="now" class="java.util.Date"/>
```

1. p名称空间注入，本质调用set

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	 xmlns:p="http://www.springframework.org/schema/p"
	 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	 xsi:schemaLocation=" http://www.springframework.org/schema/beans
	 http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="accountService"
		class="com.zzxx.service.impl.AccountServiceImpl4"
		p:name="test" p:age="21" p:birthday-ref="now"/>
	<bean id="now" class="java.util.Date"/>
</beans>
```

1. Spring表达式注入，本质调用set

   调用其他对象的属性值

```xml
<bean id="accountService" class="com.zzxx.service.impl.AccountServiceImpl">
<property name="name" value="test"/>
<property name="age" value="21"/>
<property name="birthday" ref="now"/>
</bean>
<bean name="accountService2"
	class="com.zzxx.service.impl.AccountServiceImpl">
	<property name="name" value="#{accountService.name}"/>
</bean>
```

集合属性注入方式

```xml
<!-- 注⼊集合数据
 List 结构的:
 array,list,set
 Map 结构的:
 map,entry,props,prop
-->
<bean id="accountService" class="com.zzxx.service.impl.AccountServiceImpl">
<!-- 在注⼊集合数据时，只要结构相同，标签可以互换 -->
<!-- 给数组注⼊数据 -->
	<property name="myStrs">
		<array>
			<value>AAA</value>
			<value>BBB</value>
			<value>CCC</value>
		</array>
	</property>
	<!-- 注⼊ list 集合数据 -->
		<property name="myList">
			<list>
				<value>AAA</value>
				<value>BBB</value>
				<value>CCC</value>
			</list>
		</property>
	<!-- 注⼊ set 集合数据 -->
		<property name="mySet">
			<set>
				<value>AAA</value>
				<value>BBB</value>
				<value>CCC</value>
			</set>
		</property>
	<!-- 注⼊ Map 数据 -->
		<property name="myMap">
			<map>
				<entry key="testA" value="aaa"></entry>
				<entry key="testB">
					<value>bbb</value>
				</entry>
			</map>
		</property>
		<!-- 注⼊ properties 数据 -->
		<property name="myProps">
			<props>
				<prop key="testA">aaa</prop>
				<prop key="testB">bbb</prop>
			</props>
		</property>
</bean>
```
