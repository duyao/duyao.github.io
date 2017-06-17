title: Declarative Transaction
date: 2015-10-24 14:48:02
tags: spring
categories: note

---

## Introduction

Declarative transaction management approach allows you to manage the transaction with the help of configuration instead of hard coding in your source code. This means that you can separate transaction management from the business code. 

You only use annotations or XML based configuration to manage the transactions. The bean configuration will specify the methods to be transactional.

SEE MORE:[Declarative Transaction](http://www.tutorialspoint.com/spring/declarative_management.htm "http://www.tutorialspoint.com/spring/declarative_management.htm")

<!--more-->

## Declarative transaction VS Programmatic Transaction

Declarative transaction management is preferable over programmatic transaction management though it is less flexible than programmatic transaction management, which allows you to control transactions through your code. But as a kind of crosscutting concern, declarative transaction management can be modularized with the AOP approach. Spring supports declarative transaction management through the Spring AOP framework.


## Steps

Here are the steps associated with declarative transaction:

- We use `<tx:advice />` tag, which creates a transaction-handling advice and same time we define a `pointcut` that matches all methods we wish to make transactional and reference the transactional advice.

- If a method name has been included in the transactional configuration then created advice will begin the transaction before calling the method.

- Target method will be executed in a `try / catch` block.

- If the method finishes normally, the AOP advice commits the transaction successfully otherwise it performs a rollback.


## Codes

`StudentJDBCTemplate.java`

```java
public class StudentJDBCTemplate implements StudentDAO {

	private JdbcTemplate jdbcTemplateObject;

	@Override
	public void setDataSource(DataSource ds) {
		// TODO Auto-generated method stub
		this.jdbcTemplateObject = new JdbcTemplate(ds);
	}

	@Override
	public void create(String name, Integer age, Integer marks, Integer year) {
		try {
			String SQL1 = "insert into Student (name, age) values (?, ?)";
			jdbcTemplateObject.update(SQL1, name, age);

			// Get the latest student id to be used in Marks table
			String SQL2 = "select max(id) from Student";
			int sid = jdbcTemplateObject.queryForObject(SQL2, Integer.class);

			String SQL3 = "insert into Marks(sid, marks, year) "
					+ "values (?, ?, ?)";
			jdbcTemplateObject.update(SQL3, sid, marks, year);

			System.out.println("Created Name = " + name + ", Age = " + age);
			// to simulate the exception.
			throw new RuntimeException("simulate Error condition");

		} catch (DataAccessException e) {
			System.out.println("Error in creating record, rolling back");
			throw e;
		}
	}

	@Override
	public List<StudentMarks> listStudents() {
		//pay attention to the query sentence
		String SQL = "select * from Student, Marks where Student.id=Marks.sid";

		List<StudentMarks> studentMarks = jdbcTemplateObject.query(SQL,
				new StudentMarksMapper());
		return studentMarks;
	}

}
```
`bean.xml`

```xml
<!-- don't forget the DataSource -->
<bean id="dataSource"
	class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	<property name="driverClassName" value="com.mysql.jdbc.Driver" />
	<property name="url" value="jdbc:mysql://localhost:3306/stu" />
	<property name="username" value="root" />
	<property name="password" value="mysql" />
</bean>


<!-- similarly, don't forget the PlatformTransactionManager -->
<bean id="transactionManager"
	class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>

<!-- this is the service object that we want to make transactional -->
<bean id="studentJDBCTemplate" class="com.dy.stu.StudentJDBCTemplate">
	<property name="dataSource" ref="dataSource"></property>
</bean>

 

<!-- the transactional advice (what 'happens'; see the <aop:advisor/> bean 
	below) -->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<!-- methods use the default transaction settings (see below) -->
		<tx:method name="create" />
	</tx:attributes>
</tx:advice>

<aop:config>
	<aop:pointcut id="createOperation"
		expression="execution(* com.dy.stu.StudentJDBCTemplate.create(..))" />
	<aop:advisor advice-ref="txAdvice" pointcut-ref="createOperation" />
</aop:config>
```

