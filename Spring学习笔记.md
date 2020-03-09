# Spring容器

在SpringIOC容器读取Bean配置创建Bean之前，必须对它进行实例化。只有在容器实例化后，才可以从IOC容器里获取Bean实例并使用。

Spring提供了两种类型的IOC容器实现：

1. BeanFactory：IOC容器的基本实现，是Spring框架的基础设施，面向Spring本身
2. ApplicationContext：提供了更多的高级特性，是BeanFactory的子接口 ，面向的是使用Spring框架的开发者，几乎所有的应用场合都直接使用ApplicationContext而非底层的BeanFactory

这两种实现的配置方式是一样的。

# ApplicationContext

ApplicationContext有两个主要的实现类：

1. ClassPathXmlApplicationContext：从类路径下加载配置文件
2. FileSystemXmlApplicationContext：从文件系统中加载配置文件

# 如何配置一个Bean

```xml
<bean id="helloworld" class="com.wwj.springdemo.HelloWorld">
</bean>
```

通过bean节点进行配置，其中id表示该Bean的唯一标识，class表示该Bean的全类名；

# 如何从IOC容器中获取一个Bean

要想从IOC容器中获取一个Bean，常见的有两种方式：

1. 通过id获取
2. 通过类型获取

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("application.xml");
HelloWorld helloWorld = (HelloWorld) ctx.getBean("helloworld");
```

这种方式即为通过id获取，若想通过类型获取，则应写为：

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("application.xml");
HelloWorld helloWorld = ctx.getBean(HelloWorld.class);
```

需要注意的是，通过类型获取Bean有一定的局限性，当IOC容器中存在多个类型相同的Bean时，容器将无法判断你想要哪个Bean从而抛出异常。

而通过id获取Bean则没有这样的问题，因为每个配置的Bean都对应着一个唯一的id。

# 属性注入

属性注入即通过setter方法注入Bean的属性值或依赖的对象。

```xml
<bean id="helloworld" class="com.wwj.springdemo.HelloWorld">
	<property name="name" value="spring"></property>
</bean>
```

属性注入使用property节点，其中name表示属性名，value表示属性值。

# 构造方法注入

构造方法注入即通过构造方法注入Bean的属性值或依赖的对象，它保证了Bean实例在实例化后就可以使用。

```xml
<bean id="car" class="com.wwj.springdemo.Car">
	<constructor-arg value="Audi"></constructor-arg>
	<constructor-arg value="ShangHai"></constructor-arg>
	<constructor-arg value="300000"></constructor-arg>
</bean>
```

若是这样配置，Spring将按照顺序将属性值依次注入到Bean中，你也可以指定注入属性值的先后顺序。

比如我想将Audi注入到第二个属性，ShangHai注入到第一个属性，你就可以这样：

```xml
<bean id="car" class="com.wwj.springdemo.Car">
	<constructor-arg value="Aui" index="1"></constructor-arg>
	<constructor-arg value="ShangHai" index="0"></constructor-arg>
	<constructor-arg value="300000" index="2"></constructor-arg>
</bean>
```

但很显然，这样是有缺陷的，比如这样的一个类：

```java
public class Car {
	
	private String brand;
	private String corp;
	private int price;
	private double maxSpeed;
	
    public Car(String brand, String corp,double maxSpeed) {
		this.brand = brand;
		this.corp = corp;
		this.maxSpeed = maxSpeed;
	}

	public Car(String brand, String corp, int price) {
		this.brand = brand;
		this.corp = corp;
		this.price = price;
	}
}
```

在这个类中有两个带参的构造方法，如果你还按照刚刚的方式进行配置，比如这样：

```xml
<bean id="car" class="com.wwj.springdemo.Car">
	<constructor-arg value="Aui"></constructor-arg>
	<constructor-arg value="ShangHai"></constructor-arg>
	<constructor-arg value="300000"></constructor-arg>
</bean>
```

在这段配置中，我们的目的是注入车的品牌为奥迪，厂家在上海，价格为30万，但当你获取这个Bean并输出：

```cmd
Car [brand=Aui, corp=ShangHai, price=0, maxSpeed=300000.0]
```

虽然程序并没有报错，但这样的结果也不是我们想看到的，若是想避免这样的问题，可以指定每个参数的类型：

```xml
<bean id="car" class="com.wwj.springdemo.Car">
	<constructor-arg value="Audi" type="java.lang.String"></constructor-arg>
	<constructor-arg value="ShangHai" type="java.lang.String"/>
	<constructor-arg value="300000" type="int"></constructor-arg>
</bean>
```

这样就不会有问题了。

# 属性注入细节

刚才大致地介绍了属性注入和构造方法注入的步骤和一些注意事项，下面来了解一下属性注入的细节。

## 处理特殊字符

不管是属性注入，还是构造方法注入，均可以通过value属性进行注入，这个刚刚已经介绍了，你还可以通过value子节点进行注入，比如：

```xml
<bean id="car" class="com.wwj.springdemo.Car">
	<constructor-arg value="Audi" type="java.lang.String"></constructor-arg>
	<constructor-arg value="ShangHai" type="java.lang.String"/>
	<constructor-arg type="int">
		<value>300000</value>
	</constructor-arg>
</bean>
```

它的作用是用来处理特殊字符的，比如：

```xml
<bean id="car" class="com.wwj.springdemo.Car">
	<constructor-arg value="Audi" type="java.lang.String"></constructor-arg>
	<constructor-arg type="java.lang.String">
		<value><![CDATA[<ShangHai>]]></value>
	</constructor-arg>
	<constructor-arg type="int">
		<value>300000</value>
	</constructor-arg>
</bean>
```

在xml文件中，```<>```属于特殊字符，你若想把它注入到Bean中，就需要借助```<![CDATA[]]>```来实现。

## 引用其它Bean

组成应用程序的Bean经常需要相互协作以完成应用程序的功能，要使Bean能够相互访问，就必须在配置文件中指定对Bean的引用。

定义一个Person类：

```java
public class Person {
	
	private String name;
	private int age;
	private Car car;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public Car getCar() {
		return car;
	}
	public void setCar(Car car) {
		this.car = car;
	}
}
```

在该类中有一个成员属性Car，而Car又是一个Bean，问题在于如何将一个Bean注入到另一个Bean的属性中。

```xml
<bean id="person" class="com.wwj.springdemo.Person">
	<property name="name" value="Tom"></property>
	<property name="age" value="24"></property>
	<property name="car" ref="car"></property>
</bean>
```

前面我们已经配置了一个id为car的Bean，在配置Person的时候，只需要将car作为属性值注入到该Bean的car属性即可，因为是一个引用类型，所以需要使用ref属性进行注入而非value(ref中的值为需要注入的Bean的id)，这里也可以使用ref子节点进行注入。

你也可以在属性或构造器里包含一个Bean的声明，这样的Bean称为内部Bean，比如：

```xml
<bean id="person" class="com.wwj.springdemo.Person">
	<property name="name" value="Tom"></property>
	<property name="age" value="24"></property>
	<property name="car">
		<bean class="com.wwj.springdemo.Car">
			<constructor-arg value="Ford"></constructor-arg>
			<constructor-arg value="Beijing"></constructor-arg>
			<constructor-arg value="200000"></constructor-arg>
		</bean>
	</property>
</bean>
```

该内部Bean是无法提供给外界使用的，所以id属性也就没有意义了，可以省略。

Spring还提供了级联属性注入值，比如：

```xml
<bean id="person" class="com.wwj.springdemo.Person">
	<property name="name" value="Tom"></property>
	<property name="age" value="24"></property>
	<property name="car" ref="car"></property>
	<property name="car.maxSpeed" value="250"/>
</bean>
```

这里通过car.maxSpeed为Bean的maxSpeed属性赋值(通过级联属性进行注入之前需要先初始化)。

## 注入集合类型

修改一下Person类的结构：

```java
public class Person {
	
	private String name;
	private int age;
	private List<Car> cars;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public List<Car> getCars() {
		return cars;
	}
	public void setCars(List<Car> cars) {
		this.cars = cars;
	}
}
```

在该类中有一个集合类型的成员变量，在配置的时候如何对该属性进行注入呢？

```xml
<bean id="personList" class="com.wwj.springdemo.collection.Person">
	<property name="name" value="Mike"></property>
	<property name="age" value="21"></property>
	<property name="cars">
		<list>
			<ref bean="car"/>
			<ref bean="car2"/>
			<ref bean="car3"/>
		</list>
	</property>
</bean>
```

其实很简单，首先你需要在配置文件中配置Car，然后通过list子节点注入集合，再通过ref子节点指定集合中的子元素(ref子节点中的bean属性值为对应的Bean的id)。

数组的定义和List一样，都使用list子节点，而Set集合的定义方式和List相同，唯一不同的是Map集合。，Map集合的定义如下：

```xml
<bean id="person" class="com.wwj.springdemo.collection.person">
	<property name="name" value="Jack"></property>
	<property name="age" value="30"></property>
	<property name="cars">
		<map>
			<entry key="First" value-ref="car"></entry>
			<entry key="Second" value-ref="car2"></entry>
		</map>
	</property>
</bean>
```

这里使用map子节点定义Map集合，再通过entry子节点定义集合中的子元素，其中key表示键，value-ref表示值引用。

## Properties注入

Properties是Map的子类，所以配置方式其实跟Map没有什么区别，但因为Properties的使用还是比较频繁的，所以单独拿出来介绍一下。

比较常见的使用场景便是JDBC的属性配置，先定义一个DataSource类：

```java
public class DataSource {
	
	private Properties properties;

	public Properties getProperties() {
		return properties;
	}

	public void setProperties(Properties properties) {
		this.properties = properties;
	}

	@Override
	public String toString() {
		return "DataSource [properties=" + properties + "]";
	}
}
```

在配置文件中就可以通过props节点进行注入：

```xml
<bean id="dataSource" class="com.wwj.springdemo.collection.DataSource">
	<property name="properties">
		<props>
			<prop key="user">root</prop>
			<prop key="password">123456</prop>
			<prop key="jdbcUrl">jdbc:mysql:///test</prop>
			<prop key="driverClass">com.mysql.jdbc.driver</prop>
		</props>
	</property>
</bean>
```

以上的一些配置都是在Bean的内部进行的，即这些数据只能提供给当前Bean而无法提供给外部Bean使用，为此，Spring提供了一种方式将集合类型抽取到外部供其它Bean使用：

```xml
<util:list id="cars">
	<ref bean="car"/>
	<ref bean="cars"/>
</util:list>
	
<bean id="person" class="com.wwj.springdemo.collection.Person">
	<property name="name" value="Rose"></property>
	<property name="age" value="25"></property>
	<property name="cars" ref="cars"></property>
</bean>
```

需要注意的是，必须在beans根节点里添加util schema定义：

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">
```

## 使用p命名空间

为了简化XML文件的配置，越来越多的XML文件采用属性而非子元素配置信息，Spring从2.5版本开始引入一个新的命名空间p，可以通过bean节点元素属性的方式配置Bean的属性，使用p命名空间后，基于XML的配置方式将进一步简化，比如：

```xml
<bean id="person" class="com.wwj.springdemo.collection.Person" p:name="Jery" p:age="35" p:cars-ref="cars"></bean>
```

# 自动装配

掌握了如何配置Bean之后，我们发现一个问题，就是Bean与Bean之间的关系都需要我们手动建立联系，为此，Spring提供了一种自动装配Bean的方式，我们来了解一下。

SpringIOC容器可以自动装配Bean，需要做的仅仅是在bean的autowire属性里指定自动装配的模式，装配模式有以下几种：

1. byType：根据类型自动装配。若IOC容器中有多个与目标Bean类型一致的Bean，在这种情况下，Spring将无法判定哪个Bean最适合该属性，所以不能执行自动装配
2. byName：根据名称自动装配。必须将目标Bean的名称和属性名设置为完全一致
3. constructor：根据构造器自动装配。当Bean中存在多个构造器时，该方式将会很复杂，不推荐使用

现在有这样一个Person类：

```java
public class Person {
	
	private String name;
	private Address address;

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Address getAddress() {
		return address;
	}
	public void setAddress(Address address) {
		this.address = address;
	}
}
```

该类中的成员变量Address是另外一个Bean：

```java
public class Address {
	
	private String city;
	private String street;
	
	public String getCity() {
		return city;
	}
	public void setCity(String city) {
		this.city = city;
	}
	public String getStreet() {
		return street;
	}
	public void setStreet(String street) {
		this.street = street;
	}
}
```

如果让大家按照传统的方式手动引用Bean，相信大家都会写，配置如下：

```xml
<bean id="address" class="com.wwj.springdemo.autowire.Address">
	<property name="city" value="BeiJing"></property>
	<property name="street" value="XuanWu"></property>
</bean>

<bean id="person" class="com.wwj.springdemo.autowire.Person">
	<property name="name" value="Tom"></property>
	<property name="address" ref="address"></property>
</bean>
```

那么如何实现自动装配呢？即让需要引用的Bean自动注入到另一个Bean的属性中呢？修改一下配置文件：

```xml
<bean id="address" class="com.wwj.springdemo.autowire.Address">
	<property name="city" value="BeiJing"></property>
	<property name="street" value="XuanWu"></property>
</bean>

<bean id="person" class="com.wwj.springdemo.autowire.Person" autowire="byName">
	<property name="name" value="Tom"></property>
</bean>
```

现在我们就不需要手动引用Bean了，而是在bean节点中配置一个autowire属性，它有三个值，分别有什么作用刚刚已经介绍了。比如这里指定的是byName，则IOC容器将根据名称自动装配，因为这里配置的Address类id为address，而Person类中的setter方法名为setAddress，两者对应，才能实现自动装配。

另外两种方式原理类似，不作讨论。

虽然自动装配能够帮助我们减少一些配置代码，但缺点也是很明显的，主要有以下几点：

1. 在Bean配置文件里设置autowire属性进行自动装配将会装配Bean的所有属性，然而，若只是希望装配个别属性时，autowire属性就不够灵活了
2. autowire属性要么根据类型自动装配，要么根据名称自动装配，二者不能混合使用

一般情况下，在实际的项目中很少使用自动装配功能，因为和自动装配功能所带来的好处比起来，明确清晰的配置文档更有说服力一些。

# Bean之间的关系

在Spring中Bean之间有两种关系：

1. 继承
2. 依赖

分别介绍一下。

## 继承

Spring中允许继承Bean的配置，被继承的Bean称为父Bean，继承这个父Bean的Bean称为子Bean；子Bean从父Bean中继承配置，包括Bean的属性配置；子Bean也可以覆盖从父Bean中继承过来的配置；父Bean可以作为配置模板，也可以作为Bean实例，若只想把父Bean作为模板，可以设置bean节点的abstract属性为true，这样Spring将不会实例化该Bean；也可以忽略父Bean中的class属性，让子Bean指定自己的类，而共享相同的属性配置，但此时父Bean的abstract属性值必须为true。

需要注意的是，并不是bean节点中的所有属性都会被继承，比如：autowire、abstract等是不会被子Bean继承的。

在配置文件中使用bean节点中的parent属性指定需要继承的Bean：

```xml
<bean id="address" abstract="true">
	<property name="city" value="BeiJing"></property>
	<property name="street" value="XuanWu"></property>
</bean>

<bean id="address2" class="com.wwj.springdemo.autowire.Address" parent="address">
	<property name="street" value="WuDaoKou"></property>
</bean>
```

## 依赖

Spring允许开发者通过depends-on属性设定Bean前置依赖的Bean，前置依赖的Bean会在本Bean实例化之前创建好，如果前置依赖于多个Bean，则可以通过逗号、空格的方式配置Bean的名称。比如：

```xml
<bean id="person" class="com.wwj.springdemo.autowire.Person" depends-on="address">
	<property name="name" value="Tom"></property>
</bean>
```

这个依赖是什么意思呢？看这段配置，bean节点下的属性depends-on，其值为address，若IOC容器中找不到一个id为address的Bean，则抛出异常，也就是说，该person依赖于address。

# Bean的作用域

配置文件中配置的Bean都在SpringIOC容器中有对应的作用域，可以通过bean节点中的scope属性设置作用域，若不设置，则默认为singleton。

这里介绍使用较为频繁的两种作用域：

1. singleton：容器初始化时创建Bean实例，在整个容器的生命周期中有且只有一个实例
2. prototype：容器初始化时不创建Bean实例，而在每次获取Bean时创建一个新的实例

配置如下：

```xml
<bean id="car" class="com.wwj.springdemo.Car" scope="prototype">
	<property name="brand" value="Audi"></property>
	<property name="price" value="300000"></property>
</bean>
```

# 使用外部属性文件

在配置文件中配置Bean时，有时需要在Bean的配置里混入系统部署的细节信息(比如：文件路径、数据源配置信息等)，而这些部署细节实际上需要和Bean配置相分离。

基于此，Spring提供了一个PropertyPlaceHolderConfigurer的BeanFactory后置处理器，该处理器允许开发者将Bean配置的部分内容移植到属性文件中，可以在Bean配置文件中使用形式为${var}的变量，PropertyPlaceHolderConfigurer从属性文件中加载属性，并使用这些属性来替换变量。

Spring来允许在属性文件中使用{propName}，以实现属性之间的相互作用。

比如配置JDBC的数据源信息，这里以C3P0数据库连接池为例，先导入C3P0的jar包和mysql的驱动，并作如下配置：

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="user" value="root"></property>
	<property name="password" value="123456"></property>
	<property name="driverClass" value="com.mysql.jdbc.Driver"></property>
	<property name="jdbcUrl" value="jdbc:mysql:///test"></property>
</bean>
```

这样虽然也能实现功能，但是将这些信息写死在Spring的配置文件中显然不是一个好办法，当你需要更改应用的运行环境时，你还得在Spring的配置文件中寻找数据源的配置信息并作修改。为了使操作更加方便，我们可以将这些信息移植出去，先创建一个db.properties文件：

```xml
user=root
password=123456
driverClass=com.mysql.jdbc.Driver
jdbcUrl=jdbc:mysql:///test
```

接下来我们就需要在Spring的配置文件中引用这些属性值，在这之前还需要将属性文件导入：

```xml
<context:property-placeholder location="classpath:db.properties"/>

<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="user" value="${user}"></property>
	<property name="password" value="${password}"></property>
	<property name="driverClass" value="${driverClass}"></property>
	<property name="jdbcUrl" value="${jdbcUrl}"></property>
</bean>
```

需要导入context命名空间，location属性值为属性文件路径。

# SpEL

SpEL，即：Spring表达式语言，是一中支持运行时查询和操作对象的强大的表达式语言，语法类似于EL。

SpEL使用#{...}作为定界符，所有在大括号中的字符都将被认为是SpEL，通过SpEL可以实现：

1. 通过Bean的d对Bean进行引用
2. 调用方法以及引用对象中的属性
3. 计算表达式的值
4. 正则表达式的匹配

## 为属性赋字面值

比如：

```xml
<property name="count" value="#{5}"></property>
<property name="price" value="#{12.34}"></property>
<property name="capacity" value="#{1e4}"></property>
<property name="name" value="#{"tom"}"></property>
<property name="enabled" value="true"></property>
```

但这样意义不大，简直是多此一举。

## 引用Bean、属性和方法

我们知道引用Bean可以使用ref属性，你也可以使用SpEL引用Bean，而且功能比ref更加丰富，比如：

```xml
<!-- 引用其它Bean -->
<property name="car" value="#{car}"></property>
<!-- 引用其它Bean的属性 -->
<property name="price" value="#{car.price}"></property>
<!-- 调用其它Bean的方法 -->
<property name="print" value="#{car.toString()}"></property>
<!-- 链式调用 -->
<property name="suffix" value="#{car.toString().toUpperCase()}"></property>
```

## 计算表达式的值

```xml
<!-- 加、减、乘、除 -->
<property name="add" value="#{car.price + 20000}"></property>
<property name="subtract" value="#{car.price - 20000}"></property>
<property name="multiply" value="#{2 * T(java.lang.Math).PI * circle.radius}"></property>
<property name="divide" value="#{car.price / 365}"></property>
<!-- 字符串拼接 -->
<property name="name" value="#{performer.firstName + ' ' + performer.lastName}"></property>
<!-- 比较运算符 -->
<property name="equal" value="#{car.price == 200000}"></property>
<property name="hasCapacity" value="#{car.price le 100000}"></property>
```

通过T()可以调用一个类的静态方法或者静态属性，比如这里用T()

## 正则表达式的匹配

```xml
<property name="email" value="#{admin.email matches '[a-zA-Z0-9._%+-] + @[a-zA-Z0-9.-] + \\.[a-zA-Z]{2,4}'}"></property>
```

# Bean的生命周期

SpringIOC容器可以管理Bean的生命周期，Spring允许在Bean的生命周期的特定点执行指定的任务。

SpringIOC容器对Bean的生命周期进行管理的过程：

* 通过构造器或工厂方法创建Bean的实例
* 为Bean的属性设置值和对其它Bean的引用
* 调用Bean的初始化方法
* 当容器关闭时，调用Bean的销毁方法

可以在Bean的配置中设置init-method和destroy-method属性，为Bean指定初始化和销毁方法。

定义一个类：

```java
public class Car {
	
	private String brand;
	
	public Car() {
		System.out.println("Car's Constructor...");
	}
	
	public void setBrand(String brand) {
		System.out.println("Car's setBrand...");
		this.brand = brand;
	}
	
	public void init() {
		System.out.println("Car's init...");
	}
	
	public void destroy() {
		System.out.println("Car's destroy...");
	}
}
```

然后在配置文件中进行配置：

```xml
<bean id="car" class="com.wwj.springdemo.cycle.Car" init-method="init" destroy-method="destroy">
	<property name="brand" value="BMW"></property>
</bean>
```

初始化方法和销毁方法的方法名可以任意，但是在配置时一定要与类中的方法名对应，此时测试一下便可得知Bean的生命周期，测试代码：

```java
public static void main(String[] args) {
	ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("beans-cycle.xml");
	Car car = (Car) ctx.getBean("car");
	System.out.println(car);
	ctx.close();
}
```

运行结果：

```cmd
Car's Constructor...
Car's setBrand...
Car's init...
com.wwj.springdemo.cycle.Car@5c3bd550
三月 08, 2020 3:14:04 下午 org.springframework.context.support.AbstractApplicationContext doClose
信息: Closing org.springframework.context.support.ClassPathXmlApplicationContext@6193b845: startup date [Sun Mar 08 15:14:04 CST 2020]; root of context hierarchy
Car's destroy...
```

这里为了能够关闭IOC容器，使用了ClassPathXmlApplicationContext类来进行测试，因为close()方法封装到了ConfigurableApplicationContext接口，而ClassPathXmlApplicationContext是该接口的实现类。

## Bean的后置处理器

Bean后置处理器允许在调用初始化方法前后对Bean进行额外的处理，Bean后置处理器对IOC容器中的所有Bean实例逐一处理，而非单一处理，其典型应用是：检查Bean属性的正确性或根据特定的标准更改Bean的属性，Bean后置处理器使得Bean的生命周期变得更加细致。

定义一个类实现BeanPostProcessor接口：

```java
public class MyBeanPostProcessor implements BeanPostProcessor {

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("after");
		return bean;
	}

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		System.out.println("before");
		return bean;
	}
}
```



接着修改配置文件：

```xml
<bean class="com.wwj.springdemo.cycle.MyBeanPostProcessor"></bean>

<bean id="car" class="com.wwj.springdemo.cycle.Car" init-method="init" destroy-method="destroy">
	<property name="brand" value="BMW"></property>
</bean>
```

只需要在配置文件中新增一个后置处理器的配置即可，该Bean无需指定id，IOC容器会自动寻找到该Bean并对所有Bean起作用。

重新运行一下测试代码，Bean的生命周期就会发生变化：

```cmd
Car's Constructor...
Car's setBrand...
before
Car's init...
after
com.wwj.springdemo.cycle.Car@6a4f787b
三月 08, 2020 3:31:12 下午 org.springframework.context.support.AbstractApplicationContext doClose
信息: Closing org.springframework.context.support.ClassPathXmlApplicationContext@6193b845: startup date [Sun Mar 08 15:31:12 CST 2020]; root of context hierarchy
Car's destroy...
```

# 通过工厂方法获取Bean

通过工厂方法也是获取Bean的一种方式，工厂方法又分为：

1. 静态工厂方法
2. 实例工厂方法

## 静态工厂方法

调用静态工厂方法获取Bean是将对象创建的过程封装到静态方法中，当客户端需要对象时，只需要简单地调用静态方法，而不用关心创建对象的细节。

要声明通过静态工厂方法创建的Bean，需要在Bean的class属性里指定拥有该工厂的方法的类，同时在factory-method属性里指定工厂方法的名称。最后，使用constrctor-arg元素为该方法传递方法参数。

 先定义一个静态工厂类：

```java
public class StaticCarFactory {

	private static Map<String, Car> cars = new HashMap<String, Car>();
	
	static {
		cars.put("Audi", new Car("Audi", 300000));
		cars.put("BMW",new Car("BMW", 500000));
	}
	
	public static Car getCar(String brand) {
		return cars.get(brand);
	}
}
```

该类有一个静态方法getCar()，在不需要创建这个工厂类的情况下，我们就能通过该静态方法获取到指定的Car实例，在配置文件中配置一下：

```xml
<bean id="car" class="com.wwj.springdemo.factory.StaticCarFactory" factory-method="getCar">
	<constructor-arg value="BMW"></constructor-arg>
</bean>
```

配置好后，直接通过id即可获取到Car实例：

```java
Car car = (Car) ctx.getBean("car");
```

## 实例工厂方法

实例工厂方法是将对象的创建过程封装到另外一个对象实例的方法里，当客户端需要请求对象时，只需要简单地调用一下该实例方法而无需关心对象的创建细节。

要声明通过实例工厂方法创建的Bean，需要在bean节点的factory-bean属性里指定拥有该工厂方法的Bean，在factory-method属性里指定该工厂方法的名称，并使用constructor-arg子节点为工厂方法传递方法参数。

先定义一个实例工厂类：

```java
public class InstanceCarFactory {
	
	private Map<String, Car> cars = null;
	
	public InstanceCarFactory() {
		cars = new HashMap<String, Car>();
		cars.put("Audi", new Car("Audi", 300000));
		cars.put("BMW",new Car("BMW", 500000));
	}
	
	public Car getCar(String brand) {
		return cars.get(brand);
	}
}
```

该类有一个成员方法getCar()，实例工厂与静态工厂的区别就是，实例工厂需要实例化工厂类才能获取到Car，静态工厂则不需要。

配置一下：

```xml
<!-- 配置实例工厂 -->
<bean id="carFactory" class="com.wwj.springdemo.factory.InstanceCarFactory"></bean>
	
<bean id="car" factory-bean="carFactory" factory-method="getCar">
	<constructor-arg value="Audi"></constructor-arg>
</bean>
```

然后获取Car实例：

```java
Car car = (Car) ctx.getBean("car");
```

# 通过FactoryBean获取Bean

我们还可以通过FactoryBean来获取Bean的实例，这是Spring为我们提供的一种方式。

首先定义一个类实现FactoryBean接口：

```java
public class CarFatoryBean implements FactoryBean<Car>{

	@Override
	public Car getObject() throws Exception {
		return new Car("BMW",500000);
	}

	@Override
	public Class<?> getObjectType() {
		return Car.class;
	}

	@Override
	public boolean isSingleton() {
		return true;
	}
}
```

getObject()方法返回创建的实例，getObjectType()方法返回实例类型，isSingleton()方法返回一个布尔值，标识是否单例。

配置一下：

```xml
<bean id="car" class="com.wwj.springdemo.factorybean.CarFatoryBean"></bean>
```

这样就可以通过id获取Car实例了：

```java
Car car = (Car) ctx.getBean("car");
```

# 通过注解配置Bean

前面介绍的都是基本XML文件的配置，当Bean类越来越多，Bean与Bean之间的关系越来越复杂的时候，XML文件就会显得非常臃肿，此时注解的好处就体现出来了，一起来了解一下。

## 组件扫描

Spring能够从classpath(类路径)下自动扫描，侦测和实例化具有特定注解的组件，特定组件包括：

1. @Component：基本注解，标识了一个受Spring管理的组件
2. @Respository：标识持久层组件
3. @Service：标识服务层组件
4. @Controller：标识控制层组件

对于扫描到的组件，Spring有默认的命名策略：使用非限定类名，第一个字母小写；也可以在注解中通过value属性值设定组件的名称。

这些注解虽然对应着每个业务层次的组件，你也可以随意地使用这些注解，因为Spring是无法知道你的类到底是持久层、服务层还是控制层的，但为了统一规范，也为了项目结构的严谨性，还是建议妥善使用这些注解，将这些注解标注到对应的组件上。

当在组件上使用了特定的注解之后，还需要在Spring的配置文件中声明context:component-scan，需要注意该节点中的属性：

* base-package：指定一个需要扫描的基类包，Spring容器将会扫描这个基类包及其子包中的所有类
* resource-pattern：如果仅希望扫描特定的类而非基类包下的所有类，可以使用该属性进行过滤
* context:include-filter：子节点，表示要包含的目标类
* context:exclude-filter：子节点，表示要排除的目标类

context:component-scan节点下可以包含若干个context:include-filter和context:exclude-filter子节点。

用法很简单，这样配置：

```xml
<context:component-scan base-package="com.wwj.springdemo.annotation"></context:component-scan>
```

Spring容器便会扫描com.wwj.springdemo.annotation包和该包下子包的所有类，如果有某个类含有特定的组件注解，Spring容器就会管理该Bean。

在该包下创建一个类：

```java
@Component
public class TestObject {
	
}
```

并用@Component标注它，则在获取该Bean的时候，该Bean的id应为testObject(首字母小写)，也可以指定其id：

```java
@Component(value = "test")
public class TestObject {
	
}
```

倘若该包下有一个子包com.wwj.springdemo.annotation.repository，若是想只扫描该子包下的类，则可以使用resource-pattern过滤：

```xml
<context:component-scan base-package="com.wwj.springdemo.annotation" resource-pattern="repository/*.class"></context:component-scan>
```

当然了，还可以使用context:include-filter和context:exclude-filter来包含或者排除目标类，比如：

```xml
<context:component-scan base-package="com.wwj.springdemo.annotation">
	<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
</context:component-scan>
```

在这段配置中，context:exclude-filter节点中的type属性值为annotation，则Spring容器会根据指定的注解进行排除，expression属性值为org.springframework.stereotype.Repository，则Spring容器会将@Repository注解标注的所有类全部排除，排除后就无法获取到这些Bean实例了。

还可以通过指定类名的方式排除指定的类，如：

```xml
<context:component-scan base-package="com.wwj.springdemo.annotation">
	<context:exclude-filter type="assignable" expression="com.wwj.springdemo.annotation.repository.UserRepository"/>
</context:component-scan>
```

指定类名后，Spring容器便会排除它，若指定的类是一个接口，则容器会排除该接口的所有实现类。

## 建立组件之间的关联关系

要想建立组件之间的关联关系，其实非常简单，当你使用了组件扫描之后，它会自动注册一个AutoWiredAnnotationBeanPostProcessor实例，该实例可以自动装配具有@Autowired、@Resource和@Inject注解的属性。

在刚才的基础上，我们来建立几个类：

```java
@Controller
public class UserController {
	
	@Autowired
	private UserService userService;
	
	public void execute() {
		System.out.println("UserController execute...");
		userService.add();
	}
}
```

```java
@Service
public class UserService {
	
    @Autowired
	private UserRepository userRepository;
	
	public void add() {
		System.out.println("UserService add...");
		userRepository.save();
	}
}
```

```java
@Repository
public class UserRepositoryImpl implements UserRepository{

	@Override
	public void save() {
		System.out.println("UserRepositoryImpl save...");
	}
}
```

测试代码：

```java
public static void main(String[] args) {
	ApplicationContext ctx = new ClassPathXmlApplicationContext("beans-annotation.xml");
	UserController userController = (UserController) ctx.getBean("userController");
	userController.execute();
}
```

运行结果：

```cmd
UserController execute...
UserService add...
UserRepositoryImpl save...
```

自动装配过程需要注意以下几点：

1. 构造器、普通字段，一切具有参数的方法都可以使用@Autowired注解
2. 默认情况下，所有使用@Autowired注解的属性都需要被设置，当Spring找不到匹配的Bean时，就会抛出异常，若某一个属性允许不被设置，可以设置@Autowired注解的required属性为false
3. 默认情况下，当IOC容器中存在多个类型相同的Bean时，通过类型的自动装配将无法工作，此时可以在@Qualifier注解里提供Bean的名称，Spring允许对方法的入参标注@Qualifier以指定注入Bean的名称
4. @Autowired注解也可以应用在数组类型的属性上，此时Spring将会把所有匹配的Bean进行自动装配
5. @Autowired注解也可以应用在集合类型的属性上，此时Spring读取该集合的类型信息，然后自动装配所有与之兼容的Bean
6. @Autowired注解用在Map集合上时，若该Map的键值为String，那么Spring将自动装配与之Map值类型兼容的Bean，此时Bean的名称作为键值



Spring还支持@Resource和@Inject注解，这两个注解和@Autowired注解的功能类似。

@Resource注解要求提供一个Bean名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为Bean的名称；@Inject注解和@Autowired注解一样也是按类型匹配注入的Bean，但没有reqired属性。

综上所述，建立使用@Autowired注解。

# SpringAOP

一个稳定的项目离不开多次版本的迭代，而在这个过程中，越来越多的非业务需求，比如：日志、用户验证等加入后，原有的业务方法急剧膨胀，每个方法在处理核心逻辑的同时还必须兼顾其它多个关注点。

以日志需求为例，只是为了满足这个单一需求，就不得不在多个模块里多次重复相同的日志代码，如果日志需求发生变化，就必须修改所有模块。

为了解决这些问题，AOP顺势而生。AOP即面向切面编程，它是一种新的方法论，是对传统OOP的补充。AOP的主要编程对象是切面。

在应用AOP编程时，仍然需要定义公共功能，但可以明确地定义这个功能在哪里，以什么方式应用，并且不必修改受影响的类，这样一来横切关注点就被模块化到特殊的对象(切面)里，AOP有以下好处：

1. 每个事物逻辑位于一个位置，代码不分散，便于维护和升级
2. 业务模块更简洁，只包含核心业务代码

比如这样的一个类：

```java
@Component("arithmeticCalculator")
public class ArithmeticCalculatorImpl implements ArithmeticCalculator {

	@Override
	public int add(int i, int j) {
		int result = i + j;
		return result;
	}

	@Override
	public int sub(int i, int j) {
		int result = i - j;
		return result;
	}

	@Override
	public int mul(int i, int j) {
		int result = i * j;
		return result;
	}

	@Override
	public int div(int i, int j) {
		int result = i / j;
		return result;
	}
}
```

该类用于实现加减乘除的运算，若是想在这些方法中添加日志打印，该如何实现呢？看AOP如何大显神通吧。

## 前置通知

首先定义一个切面：

```java
@Aspect
@Component
public class LoggingAspect {
	
	@Before("execution(public int com.wwj.springdemo.aop.impl.ArithmeticCalculator.*(int,int))")
	public void beforeMethod(JoinPoint joinPoint) {
		String methodName = joinPoint.getSignature().getName();
		List<Object> args = Arrays.asList(joinPoint.getArgs());
		System.out.println("before..." + methodName + " with" + args);
	}
}
```

使用@Component注解将该类交由IOC容器管理，使用@Aspect将该类声明为一个切面，该类中有一个beforeMethod()方法，该方法用于处理核心代码外的逻辑。

使用@Before标注该方法，说明该方法是一个前置通知，即：在指定方法之前执行，指定方法格式为```execution()```，括号内填写方法全名，这里的方法名采用了通配符```*```，这样就能使该前置通知在四个计算方法中都有效。

通过入参JoinPoint还能够获取到待执行的方法名和参数，如何获取看上面的代码就好了。

最后需要在配置文件中配置一下：

```xml
<context:component-scan base-package="com.wwj.springdemo.aop.impl"></context:component-scan>
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
<aop:config proxy-target-class="true"></aop:config>
```

通过配置```<aop:aspectj-autoproxy></aop:aspectj-autoproxy>```时通知注解起作用，并为匹配的类自动生成代理对象。

## 后置通知

后置通知顾名思义，就是在目标方法执行后执行(无论是否发生异常)，也就是说，倘若目标方法执行过程中出现了异常，后置通知方法仍然会执行。它们的用法是类似的，看代码：

```java
@After("execution(public int com.wwj.springdemo.aop.impl.ArithmeticCalculator.*(int,int))")
public void afterMethod(JoinPoint joinPoint) {
	String methodName = joinPoint.getSignature().getName();
	List<Object> args = Arrays.asList(joinPoint.getArgs());
	System.out.println("after..." + methodName + " with" + args);
}
```

## 返回通知

返回通知和后置通知类似，但又有些不同，无论目标方法是否出现异常，后置通知都会执行，而返回通知则不然，它必须要在目标方法正确执行完的基础上才会执行，有异常的话返回通知就不奏效了。

```java
@AfterReturning(value = "execution(public int com.wwj.springdemo.aop.impl.ArithmeticCalculator.*(int,int))",returning = "result")
public void afterReturningMethod(JoinPoint joinPoint,Object result) {
	String methodName = joinPoint.getSignature().getName();
	List<Object> args = Arrays.asList(joinPoint.getArgs());
	System.out.println("afterReturning..." + methodName + " with" + args + " result " + result);
}
```

实现方式也基本一样，需要注意的是返回通知是可以获取到目标方法执行的结果的，也就是可以获取到它的返回值，获取步骤：在@AfterReturning注解中添加returning属性值，属性值可以任意，比如这里填写的是result。

那么在方法的入参中就可以加上一个result的变量，该变量即为目标方法的返回值(方法的入参变量名一定要和注解中returning属性值一致)。

## 异常通知

异常通知是在目标方法执行出现异常的时候生效的，该通知能够获取到异常信息，代码如下：

```java
@AfterThrowing(value = "execution(public int com.wwj.springdemo.aop.impl.ArithmeticCalculator.*(int,int))",throwing = "ex")
public void afterThrowingMethod(JoinPoint joinPoint,Exception ex) {
	String methodName = joinPoint.getSignature().getName();
	List<Object> args = Arrays.asList(joinPoint.getArgs());
	System.out.println("afterThrowing..." + methodName + " with" + args + " exception " + ex);
}
```

用法和返回通知一样。

需要注意一个问题，该方法的入参ex代表需要抓取的异常，如果你产生的是一个数学异常，而你的方法入参却是NullPointException，这样异常通知也不会执行，方法入参一定要能够抓取到目标方法产生的异常，该通知才会生效。

## 环绕通知

环绕通知是功能最强大的一种通知，它能够实现前面所有通知的功能，看代码：

```java
@Around(value = "execution(public int com.wwj.springdemo.aop.impl.ArithmeticCalculator.*(int,int))")
public Object aroundMethod(ProceedingJoinPoint jp) {
	Object result = null;
	String methodName = jp.getSignature().getName();
	Object[] args = jp.getArgs();
	try {
		//前置通知
		System.out.println("before... " + methodName + " " + Arrays.asList(args));
		result = jp.proceed();//执行目标方法
		//返回通知
		System.out.println("afterReturning... " + methodName + " " + Arrays.asList(args) + " " + result);
	} catch (Throwable e) {
		//异常通知
		System.out.println("afterThrowing... " + methodName + " " + Arrays.asList(args) + e);
	}
	//后置通知
	System.out.println("after... " + methodName + " " + Arrays.asList(args));
	return result;
}
```

该通知方法必须携带ProceedingJoinPoint参数，而且必须有返回值，并以```jp.proceed();```为基础，在执行目标方法之前的为前置通知，在其之后的为返回通知，出现异常有异常通知，最后都会执行后置通知；从环绕通知里，大家也能清楚地知道为什么只有返回通知能够获取到目标方法的执行结果，为什么异常通知能够获取到异常信息，并且方法入参必须能够抓取到异常才能生效。

## 切面的优先级

当一个方法有多个切面起作用时，如何决定切面之间的优先级呢？

很简单，通过@Order注解，比如：

```java
@Order(1)
@Aspect
@Component
public class LoggingAspect {
    ......
}
```

@Order注解中的值越小，优先级就越高。

## 重用切面表达式

在前面的通知中，我们都需要去重复地编写通知作用的方法，其实是可以将这些重复的内容提取出来的，因为很简单，就直接贴代码了：

```java
@Aspect
@Component
public class LoggingAspect {
	
    //重用切面表达式
	@Pointcut("execution(public int com.wwj.springdemo.aop.impl.ArithmeticCalculator.*(int,int))")
	public void declareJointPointExpression() {}
	
	@Before("declareJointPointExpression()")
	public void beforeMethod(JoinPoint joinPoint) {
		......
	}
	
	@After("declareJointPointExpression()")
	public void afterMethod(JoinPoint joinPoint) {
		......
	}
	
	@AfterReturning(value = "declareJointPointExpression()",returning = "result")
	public void afterReturningMethod(JoinPoint joinPoint,Object result) {
		......
	}
	
	@AfterThrowing(value = "declareJointPointExpression()",throwing = "ex")
	public void afterThrowingMethod(JoinPoint joinPoint,Exception ex) {
		......
	}
}
```

使用@Pointcut注解来声明切面表达式，其它通知直接使用该方法名来引用即可。

## 基于XML文件的方式配置AOP

前面介绍的都是基于注解的方式，我们来了解一下如何通过XML文件实现AOP。

有前面的基础，这个其实很好理解：

```xml
<bean id="arithmeticCalculator" class="com.wwj.springdemo.aop.xml.ArithmeticCalculatorImpl">
</bean>
	
<!-- 配置切面 -->
<bean id="loggingAspect" class="com.wwj.springdemo.aop.xml.LoggingAspect">
</bean>
	
<!-- 配置AOP -->
<aop:config proxy-target-class="true">
	<!-- 配置切点表达式 -->
	<aop:pointcut expression="execution(int com.wwj.springdemo.aop.xml.ArithmeticCalculator.*(int,int))" id="pointcut"/>
	<!-- 配置切面及通知 -->
	<aop:aspect ref="loggingAspect" order="1">
		<!-- 前置通知 -->
		<aop:before method="beforeMethod" pointcut-ref="pointcut"/>
		<!-- 后置通知 -->
		<aop:after method="afterMethod" pointcut-ref="pointcut"/>
		<!-- 返回通知 -->
		<aop:after-returning method="afterReturningMethod" pointcut-ref="pointcut" returning="result"/>
		<!-- 异常通知 -->
		<aop:after-throwing method="afterThrowingMethod" pointcut-ref="pointcut" throwing="ex"/>
	</aop:aspect>
</aop:config>
```

相信我一句话不用说，大家都能明白。

# Spring中的事务

作为企业级应用程序框架，Spring在不同的事务管理API之上定义了一个抽象层，而应用程序开发人员不必了解底层的事务管理API，就可以使用Spring的事务管理机制。

Spring既支持编程式事务管理，也支持声明式事务管理，分别做一个介绍。

* 编程式事务：将事务管理代码嵌入到业务方法中来控制事务的提交和回滚，在编程式管理事务时，必须在每个事务操作中包含额外的事务管理代码
* 声明式事务：大多数情况下比编程式事务管理更好用，它将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理，事务管理作为一个横切关注点，可以通过AOP的方式模块化

举个例子：

```java
@Repository("bookShopDao")
public class BookShopDaoImpl implements BookShopDao {

	@Autowired
	private JdbcTemplate jdbcTemplate;
	
	@Override
	public int findBookPriceByIsbn(String isbn) {
		String sql = "SELECT price FROM book WHERE isbn = ?";
		return jdbcTemplate.queryForObject(sql, Integer.class, isbn);
	}

	@Override
	public void updateBookStock(String isbn) {
		//检查书的库存是否足够, 若不够, 则抛出异常
		String sql2 = "SELECT stock FROM book_stock WHERE isbn = ?";
		int stock = jdbcTemplate.queryForObject(sql2, Integer.class, isbn);
		if(stock == 0){
			throw new BookStockException("库存不足!");
		}
		
		String sql = "UPDATE book_stock SET stock = stock -1 WHERE isbn = ?";
		jdbcTemplate.update(sql, isbn);
	}

	@Override
	public void updateUserAccount(String username, int price) {
		//验证余额是否足够, 若不足, 则抛出异常
		String sql2 = "SELECT balance FROM account WHERE username = ?";
		int balance = jdbcTemplate.queryForObject(sql2, Integer.class, username);
		if(balance < price){
			throw new UserAccountException("余额不足!");
		}
		
		String sql = "UPDATE account SET balance = balance - ? WHERE username = ?";
		jdbcTemplate.update(sql, price, username);
	}

}
```

现在有这样一个实现类，这个类在做什么呢？它有三个方法，第一个方法：根据书号获取书的单价；第二个方法：根据书号更新书的库存；第三个方法：更新用户的账户余额(该类还涉及到JdbcTemplate类的使用，因为比较简单，所以不作讲解)。

我们再编写服务层代码：

```java
@Service
public class BookShopServiceImpl implements BookShopService {

	@Autowired
	private BookShopDao bookShopDao;
	
	@Override
	public void purchase(String username, String isbn) {
		//获取书的单价
		int price = bookShopDao.findBookPriceByIsbn(isbn);
		//更新书的库存
		bookShopDao.updateBookStock(isbn);
		//更新用户余额
		bookShopDao.updateUserAccount(username, price);
	}
}
```

该类中有一个购书的方法，当用户购书后，应该相信地减少书的库存和用户的余额。

到这里一个简单的应用场景就搭建好了，用户只要进行购书，就会相应地改变数据库表的值，然而这里有一个很明显的问题：当书的库存足够而用户的余额不足时，用户在购书时，因为业务代码先更新了书的库存，后更新用户余额，所以会出现购买失败而书的库存却减少的情况，原因是异常在更新用户余额时才抛出。

为此，我们需要使用事务来让这两个操作要么同时成功，要么同时失败，先来看看Spring的声明式事务。

## 声明式事务

我们需要在配置文件中进行一些简单的配置：

```xml
<!-- 引入外部属性文件 -->
<context:property-placeholder location="classpath:db.properties"/>
	
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="user" value="${root}"></property>
	<property name="password" value="${password}"></property>
	<property name="jdbcUrl" value="${jdbcUrl}"></property>
	<property name="driverClass" value="${driverClass}"></property>
</bean>
	
<!-- 配置JdbcTemplate -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
	<property name="dataSource" ref="dataSource"></property>
</bean>
	
<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>
	
<!-- 启用事务注解 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```

前面的配置相信大家都不陌生，关键就是后面的两个配置，首先配置事务管理器，并将dataSource注入，然后配置```<tx:annotation-driven transaction-manager="transactionManager"/>```用于启动事务注解，属性transaction-manager的值为事务管理器的id。

配置好后，我们修改一下服务层代码：

```java
@Service
public class BookShopServiceImpl implements BookShopService {

	@Autowired
	private BookShopDao bookShopDao;
	
	//事务注解
	@Transactional
	@Override
	public void purchase(String username, String isbn) {
		//获取书的单价
		int price = bookShopDao.findBookPriceByIsbn(isbn);
		//更新书的库存
		bookShopDao.updateBookStock(isbn);
		//更新用户余额
		bookShopDao.updateUserAccount(username, price);
	}
}
```

其实什么也没改，仅仅加了一个@Transactional注解，事务就已经启动了，就是这么简单。

## 事务的传播行为

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播，例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。

事务的传播行为可由传播属性指定，Spring定义了7种传播行为：

|   传播属性   |                             描述                             |
| :----------: | :----------------------------------------------------------: |
|   REQUIRED   | 如果有事务在运行，当前的方法就在这个事务内运行，否则，就启动一个新的事务，并在自己的事务内运行 |
| REQUIRED_NEW | 当前的方法必须启动新事务，并在自己的事务内执行，如果有事务正在运行，会先将其挂起 |
|   SUPPORTS   | 如果有事务正在运行，当前的方法就在该事务内运行，否则它可以不运行在事务中 |
| NOT_SUPPORTE |  当前的方法不应该运行在事务中，如果有运行的事务，会将其挂起  |
|  MANDATORY   | 当前的方法必须运行在事务中，如果没有正在运行的事务，就抛出异常 |
|    NEVER     |  当前的方法不应该运行在事务中，如果有运行的事务，就抛出异常  |
|    NESTED    | 如果有事务正在运行，当前的方法就应该在这个事务的嵌套事务内运行，否则， 就启动一个新的事务，并在它自己的事务内运行 |

事务的默认传播行为是REQUIRED，可以通过@Transactional注解中的propagation属性指定传播行为，比如：

```java
//事务注解
@Transactional(propagation = Propagation.REQUIRED)
@Override
public void purchase(String username, String isbn) {
    ......
}
```

当然了，你还可以设置一些事务的其它属性，比如事务的隔离级别，通过isolation属性设置即可，本文的重点是Spring，就直接略过了。

## 基于XML文件的方式配置事务

掌握了注解实现事务，XML方式实现就会很简单，它很像一个翻译的过程：

```xml
<!-- 引入外部属性文件 -->
<context:property-placeholder location="classpath:db.properties"/>
	
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
	<property name="user" value="${root}"></property>
	<property name="password" value="${password}"></property>
	<property name="jdbcUrl" value="${jdbcUrl}"></property>
	<property name="driverClass" value="${driverClass}"></property>
</bean>
	
<!-- 配置JdbcTemplate -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
	<property name="dataSource" ref="dataSource"></property>
</bean>
	
<bean id="bookShopDao" class="com.wwj.springdemo.tx.BookShopDaoImpl">
	<property name="jdbcTemplate" ref="jdbcTemplate"></property>
</bean>
	
<bean id="bookService" class="com.wwj.springdemo.tx.BookShopServiceImpl">
	<property name="bookShopDao" ref="bookShopDao"></property>
</bean>

<!-- 配置事务管理器 -->	
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>
	
<!-- 配置事务属性 -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<tx:method name="*" propagation="REQUIRED"/>
	</tx:attributes>
</tx:advice>
	
<!-- 配置事务切入点 -->
<aop:config>
	<aop:pointcut expression="execution(* com.wwj.springdemo.tx.BookShopService.*(..))" id="pointcut"/>
	<!-- 关联事务属性 -->
	<aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>
</aop:config>
```

事务的一些其它属性都可以在tx:method节点中进行配置。











