# Introduction to SQL

## CREATE TABLE
``` sql
CREATE TABLE department
(
    dept_name varchar(20),
    building varchar(15),
    budget numeric(12,2),
    primary key (dept_name)
)
CREATE TABLE course
(
     course_id		varchar(8), 
	 title			varchar(50), 
	 dept_name		varchar(20),
	 credits		numeric(2,0) check (credits > 0),
	 primary key (course_id),
	 foreign key (dept_name) references department
		on delete set null
)
CREATE TABLE instructor
(
     ID			varchar(5), 
	 name			varchar(20) not null, 
	 dept_name		varchar(20), 
	 salary			numeric(8,2) check (salary > 29000),
	 primary key (ID),
	 foreign key (dept_name) references department
		on delete set null
)
CREATE TABLE section
(
    course_id		varchar(8), 
    sec_id			varchar(8),
	semester		varchar(6)
		check (semester in ('Fall', 'Winter', 'Spring', 'Summer')), 
	year			numeric(4,0) check (year > 1701 and year < 2100), 
	building		varchar(15),
	room_number		varchar(7),
	time_slot_id		varchar(4),
	primary key (course_id, sec_id, semester, year),
	foreign key (course_id) references course
		on delete cascade,
	foreign key (building, room_number) references classroom
		on delete set null
)    
CREATE TABLE teaches
(
    ID			varchar(5), 
	course_id		varchar(8),
	sec_id			varchar(8), 
	semester		varchar(6),
	year			numeric(4,0),
	primary key (ID, course_id, sec_id, semester, year),
	foreign key (course_id,sec_id, semester, year) references section
		on delete cascade,
	foreign key (ID) references instructor
		on delete cascade
)
```

## DROP TABLE 
``` sql
DROP TABLE instructor
```
![DropTable](drop-table.png)
This deletes all tuples of  _instructor_, as well as the schema for _instructor_. After  _instructor_ is dropped, no tuples can be inserted into  _instructor_ unless it is re-created with the `CREATE TABLE` command.

## DELETE FROM 
``` sql
DELETE FROM instructor
```
This retains the relation _instructor_ but deletes all tuples in _instructor_.
![DeleteFrom](delete-from.png)

## ALTER TABLE
``` sql
ALTER TABLE instructor ADD address varchar(20)
```
![AlterTable](alter-table1.png)

``` sql
ALTER TABLE instructor drop address
```
We can drop and add attributes in a relation using `ALTER TABLE`

## QUERIES ON A SINGLE RELATION 

To find names of all instructors in the university database: 
``` sql
SELECT name FROM instructor
```
![SELECT](select-1.png)

 The instructor's name appears in the name attribute, and then we put in a `select` clause.

To find the department names of all instructors: 
``` sql
SELECT dept_name FROM instructor
```
![SELECT](select-2.png)

The result of the above query is aa relation containing all department names, which may contain duplicate values as well since more than one instructors can belong to a department. If we want to force the elimination of duplicates, we insert the keyword `distinct` after `select`. 
``` sql
SELECT distinct dept_name FROM instructor
```
![SELECT](select-3.png)

The result of the above query would contain each department name at most once. 

SQL allows us to use the keyword all to specify explicity that duplicates are not removed. 
``` sql
SELECT all dept_name FROM instructor
```
![SELECT](select-2.png)
But duplicate retention is defualt, so using `all` is not suggested. 

SQL clause may also contain arithmetic expressions involving operators like `+ , -, *, /` operating on constants or attributes of tuples. 
``` sql
SELECT ID, name, dept_name, salary*1.1 FROM instructor
```
![SELECT](select-4.png)
This query returns a relation that is the same as the `instructor` relation, except that the attribute `salary` is mulitiplied by 1.1. This does not result in any change to the `instructor` relation.

To find the names of all instructors in the Computer Science department who have salary greater than $70,000:
``` sql
SELECT name from instructor where dept_name = 'Comp. Sci.' and salary>70000
```
![SELECT](select-5.png)



## QUERIES ON MULTIPLE RELATIONS

To retrieve names of all instructors along with their department names and department building name: 

``` sql
select name, instructor.dept_name, building from instructor, department where instructor.dept_name = department.dept_name
```

Cartesian product of the `instructor` relation with `teaches` relation is as follows: 
``` sql
SELECT * FROM instructor, teaches
```
![SELECT](select-6.png)
This results in tuples(12*13 = 156) from `instructor` and `teaches` that are unrelated to each other. Instead, we would want a query involving instructor and teaches to combine a particular tuple t in instructor with only those tuples in teaches that refer to the same instructor to which t refers.
``` sql
SELECT name,course_id FROM instructor, teaches where instructor.ID = teaches.ID
```
 ![SELECT](select-7.png)

 If we wished to find only instructor names and course identifiers for instructors in the Computer Science departmnet, we could add an extra predicate to the `where` clause:
 ``` sql
SELECT name,course_id FROM instructor, teaches where instructor.ID = teaches.ID and instructor.dept_name = 'Comp. Sci.'
```
 ![SELECT](select-8.png)

 ## THE RENAME OPERATION
 
 The `as` clause appears in both the `select` and `from` clauses.
 - If we want the attribute name `name` to be replaced with the 
    name `instructor_name`, we can write:
    ``` sql
    SELECT name as instructor_name, course_id FROM instructor, teaches WHERE instructor.ID = teaches.ID
    ```
    ![SELECT](rename-1.png)

 - to find names and course_ID all instructors in the university who have taught some course.
    ``` sql
    SELECT T.name, S.course_id FROM instructor as T, teaches as S WHERE T.ID = S.ID
    ```
    ![SELECT](rename-2.png)
- To compare tuples in the same relation, we would need cartesian product of a relation with itself, and without renaming, it would be impossible to distinguish one tuple from the other. For example, to find the names of all instructors whose salary is strictly greater than atleast one instructor in the Biology department,  
     ``` sql
    SELECT distinct T.name FROM instructor as T, instructor as S WHERE T.salary > S.salary and S.dept_name = 'Biology'
    ```
    ![SELECT](rename-3.png)