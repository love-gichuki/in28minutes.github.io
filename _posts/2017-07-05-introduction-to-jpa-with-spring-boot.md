---
layout:     post
title:      Introduction to JPA using Spring Boot Data Jpa
date:       2017-07-05 12:31:19
summary:    Understand JPA and setup a simple JPA example using Spring Boot
categories: JPA, Spring Boot, Hibernate
permalink:  /introduction-to-jpa-with-spring-boot-data-jpa
---

This guide will help you understand what JPA is and setup a simple JPA example using Spring Boot.
 
## You will learn
- What is JPA?
- What is Object Relational Impedence?
- What are the alternatives to JPA? 
- What is Hibernate and How does it relate to JPA?
- How to create a simple JPA project using Spring Boot Data JPA Starter?

## References

1 hour video courses on all popular frameworks!

- Spring - [spring-framework-tutorial-for-beginners](https://in28minutes.teachable.com/p/spring-framework-tutorial-for-beginners)
- Spring MVC - [https://www.youtube.com/watch?v=BjNhGaZDr0Y](https://www.youtube.com/watch?v=BjNhGaZDr0Y)
- Spring Boot - [https://www.youtube.com/watch?v=PSP1-2cN7vM](https://www.youtube.com/watch?v=PSP1-2cN7vM)
- Eclipse - [https://www.youtube.com/watch?v=s4ShbtOHMCA](https://www.youtube.com/watch?v=s4ShbtOHMCA)
- Maven - [https://www.youtube.com/watch?v=0CFWeVgzsqY](https://www.youtube.com/watch?v=0CFWeVgzsqY)
- JUnit - [https://www.youtube.com/watch?v=o5k9NOR9lrI](https://www.youtube.com/watch?v=o5k9NOR9lrI)
- Mockito - [https://www.youtube.com/watch?v=d2KwvXQgQx4](https://www.youtube.com/watch?v=d2KwvXQgQx4)

## Tools you will need
- Maven 3.0+ is your build tool
- Your favorite IDE. We use Eclipse.
- JDK 1.8+
- In memory database H2

## What is Object Relational Impedence Mismatch?

Java is an object oriented programming language. In Java, all data is stored in objects.

Typically, Relational databases are used to store data (These days, a number of other NoSQL data stores are also becoming popular - We will stay away from them, for now).  Relational databases store data in tables.

The way we design objects is different from the way the relational databases are designed. This results in an impedence mismatch.
 - Object Oriented programming consists of concepts like encapsulation, inheritance, interfaces and polymorphism 
 - Relational databases are made up of Tables with concepts like normalization 

### Examples of Object Relational Impedence Mismatch

Lets consider a simple example - Employees and Tasks.

Each Employee can have multiple Tasks. Each Task can be shared by multiple Employees. There is a Many to Many relationship between them. Let's consider a few examples of impedence mismatch.

#### Example 1 : Task table below is mapped to Task Table. However, there are mismatches in column names.

```java
public class Task {
	private int id;

	private String desc;

	private Date targetDate;

	private boolean isDone;

	private List<Employee> employees;
}
```

```sql
 CREATE TABLE task 
  ( 
     id          INTEGER GENERATED BY DEFAULT AS IDENTITY, 
     description VARCHAR(255), 
     is_done     BOOLEAN, 
     target_date TIMESTAMP, 
     PRIMARY KEY (id) 
  ) 
```

#### Example 2 : Relationships between objects are expressed in a different way compared with relationship between tables.

Each Employee can have multiple Tasks. Each Task can be shared by multiple Employees. There is a Many to Many relationship between them.

```java
public class Employee {
   
     //Some other code
	
	private List<Task> tasks;
}

public class Task {

     //Some other code

	private List<Employee> employees;
}
```

```sql
CREATE TABLE employee 
  ( 
     id            BIGINT NOT NULL, 
     OTHER_COLUMNS
  ) 


  CREATE TABLE employee_tasks 
  ( 
     employees_id BIGINT NOT NULL, 
     tasks_id     INTEGER NOT NULL 
  ) 

  CREATE TABLE task 
  ( 
     id          INTEGER GENERATED BY DEFAULT AS IDENTITY, 
     OTHER_COLUMNS
  ) 

```

#### Example 3 : Some times multiple classes are mapped to a single table and viceversa

Objects

```java
public  class Employee {	
    //Other Employee Attributes
}

public class FullTimeEmployee extends Employee {
	protected Integer salary;
}

public class PartTimeEmployee extends Employee {
	protected Float hourlyWage;
}
```

Tables
```sql
CREATE TABLE employee 
  ( 
     employee_type VARCHAR(31) NOT NULL, 
     id            BIGINT NOT NULL, 
     city          VARCHAR(255), 
     state         VARCHAR(255), 
     street        VARCHAR(255), 
     zip           VARCHAR(255), 

     hourly_wage   FLOAT,  --PartTimeEmployee

     salary        INTEGER, --FullTimeEmployee

     PRIMARY KEY (id) 
  ) 

```

## Other approaches before JPA - JDBC, Spring JDBC & myBatis

Other approaches before JPA focused on queries and how to translate results from queries to objects.

### JDBC

- JDBC stands for Java Database Connectivity
- It used concepts like Statement, PreparedStatement and ResultSet
- In the example below, the query used is ```Update todo set user=?, desc=?, target_date=?, is_done=? where id=?```
- The values needed to execute the query are set into the query using different set methods on the PreparedStatement
- Results from the query are populated into the ResultSet. We had to write code to liquidate the ResultSet into objects.


#### Update Todo
```java
Connection connection = datasource.getConnection();

PreparedStatement st = connection.prepareStatement(
		"Update todo set user=?, desc=?, target_date=?, is_done=? where id=?");

st.setString(1, todo.getUser());
st.setString(2, todo.getDesc());
st.setTimestamp(3, new Timestamp(
		todo.getTargetDate().getTime()));
st.setBoolean(4, todo.isDone());
st.setInt(5, todo.getId());

st.execute();

st.close();

connection.close();
```

#### Retrieve a Todo
```java
Connection connection = datasource.getConnection();

PreparedStatement st = connection.prepareStatement(
		"SELECT * FROM TODO where id=?");

st.setInt(1, id);

ResultSet resultSet = st.executeQuery();


if (resultSet.next()) {

    Todo todo = new Todo();
	todo.setId(resultSet.getInt("id"));
	todo.setUser(resultSet.getString("user"));
	todo.setDesc(resultSet.getString("desc"));
	todo.setTargetDate(resultSet.getTimestamp("target_date"));
	return todo;
}

st.close();

connection.close();

return null;
```

### Spring JDBC

- Spring JDBC provides a layer on top of JDBC
- It used concepts like JDBCTemplate
- Typically needs lesser number of lines compared to JDBC as following are simplified
   - mapping parameters to queries
   - liquidating resultsets to beans


#### Update Todo
```java
jdbcTemplate
				.update("Update todo set user=?, desc=?, target_date=?, is_done=? where id=?",
						todo.getUser(), 
						todo.getDesc(),
						new Timestamp(todo.getTargetDate().getTime()),
						todo.isDone(), 
						todo.getId()
					);
```

#### Retrieve a Todo

```java
@Override
public Todo retrieveTodo(int id) {

	return jdbcTemplate.queryForObject(
			"SELECT * FROM TODO where id=?",
			new Object[] { id }, new TodoMapper());

}
```

Reusable Row Mapper

```java
// new BeanPropertyRowMapper(TodoMapper.class)
class TodoMapper implements RowMapper<Todo> {
	@Override
	public Todo mapRow(ResultSet rs, int rowNum)
			throws SQLException {
		Todo todo = new Todo();

		todo.setId(rs.getInt("id"));
		todo.setUser(rs.getString("user"));
		todo.setDesc(rs.getString("desc"));
		todo.setTargetDate(
				rs.getTimestamp("target_date"));
		todo.setDone(rs.getBoolean("is_done"));
		return todo;
	}
}	
```

### myBatis

MyBatis removes the need for manually writing code to set parameters and retrieve results. It provides simple XML or Annotation based configuration to map Java POJOs to database.

#### Update Todo and Retrieve Todo


```java
@Mapper
public interface TodoMybatisService
		extends TodoDataService {

	@Override
	@Update("Update todo set user=#{user}, desc=#{desc}, target_date=#{targetDate}, is_done=#{isDone} where id=#{id}")
	public void updateTodo(Todo todo) throws SQLException;

	@Override
	@Select("SELECT * FROM TODO WHERE id = #{id}")
	public Todo retrieveTodo(int id) throws SQLException;
}

public class Todo {

	private int id;

	private String user;

	private String desc;

	private Date targetDate;

	private boolean isDone;
}

```

### Comparison with JPA
- All the three approaches used queries.
- In big application, queries can become complex. Especially when we retrieve data from multiple tables.
- This creates a problem whenever there are changes in the structure of the database.

## How does JPA Work?

![Image](/images/JPA_01_Introduction.png "JPA Introduction")

![Image](/images/JPA_02_Architecture.png "JPA Architecuture")


