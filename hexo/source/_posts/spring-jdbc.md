title: spring_jdbc
date: 2015-10-23 20:11:55
tags: spring
categories: note
---

> `JdbcTemplate` the central framework class that manages all the database communication and exception handling.

> Instances of the JdbcTemplate class are threadsafe once configured. So you can configure a single instance of a JdbcTemplate and then safely inject this shared reference into multiple DAOs.

> A common practice when using the JdbcTemplate class is to configure a DataSource in your Spring configuration file, and then dependency-inject that shared DataSource bean into your DAO classes, and the JdbcTemplate is created in the setter for the DataSource.

reference: [jdbc_spring](http://www.tutorialspoint.com/spring/spring_jdbc_framework.htm/ "http://www.tutorialspoint.com/spring/spring_jdbc_framework.htm")
<!--more-->

---

`StudentJDBCTemplate.java`

```java

public class StudentJDBCTemplate implements StudentDao {

	//configure a DataSource
	private DataSource dataSource;
	private JdbcTemplate jdbcTemplateObject;

	@Override
	public void setDataSource(DataSource ds) {
		// TODO Auto-generated method stub
		this.dataSource = ds;
		this.jdbcTemplateObject = new JdbcTemplate(ds);

	}

	@Override
	public void create(String name, Integer age) {
		String SQL = "insert into Student (name, age) values (?, ?)";

		jdbcTemplateObject.update(SQL, name, age);
		System.out.println("Created Record Name = " + name + " Age = " + age);
		return;

	}

	@Override
	public Student getStudent(Integer id) {
		// TODO Auto-generated method stub
		String SQL = "select id,name,age from student where id = ? ";
		//.queryForObject(String sql, Object[] args, RowMapper<Student> rowMapper)
		//Query given SQL to create a prepared statement from SQL and a list of arguments to bind to the query, mapping a single result row to a Java object via a RowMapper
		Student student = jdbcTemplateObject.queryForObject(SQL, new Object[]{id}, new StudentMapper());
		return student;
	}

	@Override
	public List<Student> listStudents() {
		String SQL = "select * from student";

		List<Student> list = jdbcTemplateObject.query(SQL, new StudentMapper());

		return list;
	}

	@Override
	public void delete(Integer id) {
		// TODO Auto-generated method stub
		String SQL = "delete from Student where id = ?";
		jdbcTemplateObject.update(SQL, id);
		System.out.println("Deleted Record with ID = " + id);
		return;
	}

	@Override
	public void update(Integer id, Integer age) {
		// TODO Auto-generated method stub
		String SQL = "update Student set age = ? where id = ?";
		jdbcTemplateObject.update(SQL, age, id);
		System.out.println("Updated Record with ID = " + id);
		return;
	}

	public DataSource getDataSource() {
		return dataSource;
	}

}
```
---

`bean.xml`

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
  <property name="url" value="jdbc:mysql://localhost:3306/stu"/>
  <property name="username" value="root"/>
  <property name="password" value="mysql"/>
</bean>

<!--single instance-->
<bean id="studentJDBCTemplate" class="com.dy.stu.StudentJDBCTemplate">
	<property name="dataSource" ref="dataSource"></property>
</bean>
```
---
The `SimpleJdbcCall` class can be used to call a stored procedure with IN and OUT parameters.

see :[stored procedure ](http://www.tutorialspoint.com/spring/calling_stored_procedure.htm "http://www.tutorialspoint.com/spring/calling_stored_procedure.htm")
