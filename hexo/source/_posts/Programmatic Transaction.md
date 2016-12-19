title: Programmatic Transaction
date: 2015-10-24 21:02:27
tags: spring
categories: note
---

## Introduction
Programmatic transaction management approach allows you to manage the transaction with the help of programming in your source code. That gives you extreme flexibility, but it is difficult to maintain.

SEE MORE: [Programmatic Transaction](http://www.tutorialspoint.com/spring/programmatic_management.htm "http://www.tutorialspoint.com/spring/programmatic_management.htm")

<!--more-->

## Declarative transaction VS Programmatic Transaction

Declarative transaction management is preferable over programmatic transaction management though it is less flexible than programmatic transaction management, which allows you to control transactions through your code. But as a kind of crosscutting concern, declarative transaction management can be modularized with the AOP approach. Spring supports declarative transaction management through the Spring AOP framework.


## Steps

Let us use `PlatformTransactionManager` directly to implement programmatic approach to implement transactions. 

1. Have a instance of `TransactionDefinition` with the appropriate transaction attributes. 
For this example we will simply create an instance of `DefaultTransactionDefinition` to use the default transaction attributes.

2. Once the `TransactionDefinition` is created, start transaction by calling `getTransaction()` method, which returns an instance of `TransactionStatus`.

3. The `TransactionStatus` objects helps in tracking the current status of the transaction and finally, 
if everything goes fine, use `commit()` method of PlatformTransactionManager to commit the transaction,
otherwise use `rollback()` to rollback the complete operation.

## Code 

`StudentJDBCTemplate_programmatic.java`

```java
public class StudentJDBCTemplate_programmatic implements StudentDAO {
	private DataSource dataSource;
	private JdbcTemplate jdbcTemplateObject;
	private PlatformTransactionManager transactionManager;

	@Override
	public void setDataSource(DataSource ds) {
		// TODO Auto-generated method stub
		this.dataSource = ds;
		this.jdbcTemplateObject = new JdbcTemplate(ds);

	}

	public void setTransactionManager(
			PlatformTransactionManager platformTransactionManager) {
		this.transactionManager = platformTransactionManager;
	}

	@Override
	public void create(String name, Integer age, Integer marks, Integer year) {
		// TODO Auto-generated method stub

		TransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
		//status can commit or rollback
		TransactionStatus status = this.transactionManager
				.getTransaction(transactionDefinition);

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

			//success
			transactionManager.commit(status);

		} catch (DataAccessException e) {
			// TODO: handle exception
			System.out.println("Error in creating record, rolling back");
			
			//fail
			transactionManager.rollback(status);
			throw e;
		}

	}

	@Override
	public List<StudentMarks> listStudents() {
		String SQL = "select * from Student, Marks where Student.id=Marks.sid";

		List<StudentMarks> studentMarks = jdbcTemplateObject.query(SQL,
				new StudentMarksMapper());
		return studentMarks;
	}

}

```

`applicationContext_programmatic.xml`

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
<bean id="studentJDBCTemplate" class="com.dy.stu.StudentJDBCTemplate_programmatic">
	<property name="dataSource" ref="dataSource"></property>
	<property name="transactionManager" ref="transactionManager"></property>
</bean>

```