# Pewlett Hackard Analysis
 
## Overview of the Analysis
Pewlett Hackard, PH, would like to find out the job titles of retiring employees.  They also want to identify employess who are eligible to participate in a mentorship program.  This will help them prepare for a "silver tsunami" as current employees reach retirement age.

## Setup for Analysis
PH has provided several Human Resource files in comma-separated values, CSV, format.  To provide the information PH wants about retirees PostgresSQL is used.  First, prior to creating a database from the files, and after viewing the raw files, an entity relationship diagram, ERD, is created to map out the relationships between the files so that they can become tables within the database.<br>
The ERD for this project looks like this:  <br>
<img src="https://github.com/linb960/Pewlett-Hackard-Analysis/blob/main/EmployeeDB.png" width="400" height="360"/>
<br>
Once the ERD is complete the database is created and the schema.sql is run to populate the database with the tables.  Finally the data is imported from each csv file into it's respective table. 

## Analysis
The ERD shows that to find the job titles of retiring employees we will need to join two tables, employees and titles.   We also need to make sure we only find employees who are retiring soon so birthdates must be queried from 1952 through 1955.  Our query to do this is as follows:
```
 SELECT e.emp_no,
	e.first_name,
	e.last_name,
	t.title,
	t.from_date,
	t.to_date
INTO retirement_titles
FROM titles as t
INNER JOIN employees as e
ON (e.emp_no = t.emp_no)
WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31')
ORDER BY emp_no
```
Because some employees have had more than one title while working at PH the company needs to get the list of the employees retiring with their current title. Here is the code to run against the new retirement_titles table to get the distinct entries:
```
SELECT DISTINCT ON (emp_no) emp_no,
first_name,
last_name,
title
INTO unique_titles
FROM retirement_titles
ORDER BY emp_no, to_date DESC;
```

In addition the company wants to know who can be mentored to take over for those who are retiring soon so this query is run against three tables, employees, dept_emp, and title.  In addition we want the employees birth date and to make sure they are eligible for the mentorship program they need to be born in 1965.  Here's the query:
```
SELECT DISTINCT ON (emp_no) e.emp_no,
	e.first_name,
	e.last_name,
	e.birth_date,
	de.from_date,
	de.to_date,
	t.title	
INTO mentorship_eligibilty
FROM employees as e
INNER JOIN titles as t
ON (e.emp_no = t.emp_no)
INNER JOIN dept_employee as de
ON (e.emp_no = de.emp_no)
WHERE (de.to_date = '9999-01-01')
AND (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31');
```
## Results
#### Four Major Points from our Analysis
* 90,398 people were found to be close to retiring.  This comes from unique_titles table generated from the list of the employees retiring with their current title only.
* 1,549 people are eligible to be a part of the mentorship program if we only consider those born in 1965.
* 2 Managers, 29,414 Senior Engineers and 28,254 Senior Staff are retiring which will leave a large gap in upper level positions.  This was found by counting up just the titles from the list of the employees retiring. The full count is here:<br>
&emsp;	29414	Senior Engineer<br>
&emsp;	28254	Senior Staff<br>
&emsp;	14222	Engineer<br>
&emsp;	12243	Staff<br>
&emsp;	4502	Technique Leader<br>
&emsp;	1761	Assistant Engineer<br>
&emsp;	2	Manager<br>
* Finally there are not enough people remaining with the company to fill all the vacancies that will occur when people start retiring.

## Summary
The "silver tsunami" will create a great gap in talent in the company.  The impact on PH will be most noticable in management and senior staffing positions as 2 managers, 29,414 Senior Engineers and 28,254 Senior Staff will all be retiring soon.  The mentoring program will be very important to bring current staff and engineers up to senior positions.  

To answer this question about whether there enough qualified, retirement-ready employees in the departments to mentor the next generation of Pewlett Hackard employees all we have to do is look at the number of retirement ready empoloyees.  Since there are over 90,000 people retiring there are plenty of people to mentor the only 1,549 current employees, not retiring soon, to take part in the mentor program.

### Additional Analysis
#### Find more employees eligible to Mentor
Because there are so few employees found for the mentorship program it seems that an additional query should be run.  Since the first query looked at only those born in 1965 but retirement numbers were based on three years it might be better to run this query for 1963-65:<br>
```
SELECT DISTINCT ON (emp_no) e.emp_no,
	e.first_name,
	e.last_name,
	e.birth_date,
	de.from_date,
	de.to_date,
	t.title	
INTO mentorship_eligibilty_updated
FROM employees as e
INNER JOIN titles as t
ON (e.emp_no = t.emp_no)
INNER JOIN dept_employee as de
ON (e.emp_no = de.emp_no)
WHERE (de.to_date = '9999-01-01')
AND (e.birth_date BETWEEN '1963-01-01' AND '1965-12-31');
```
<br>
This query yields 38,401 employees eligible for the mentorship program.   A 3:1 ratio of people retiring to one person eligible to take that persons place.  Prior the program had 9 people retiring for every 1 person eligible.

#### Find out what the jobs the current employees eligible for the mentoring program
If the basis for selection into the mentorship program goes ahead with the employees born 1963 to 1965 then we can use a query to count the title of those employees to gauge if there will be enough retirement eligible personnel avaible to mentor them.  The query used is here:

```
SELECT COUNT(title), title
INTO mentorship_eligibilty_titles
FROM mentorship_eligibilty_updated
GROUP BY title
ORDER BY count DESC;
```
Our results show that the numbers are much better when compared side by side.  The Senior Staff and Technique Leaders are close to a 2:1 ratio, Engineers and Assistant Engineers are close to 1:1.  Staff is 3:1.  Managers look very good with these numbers with 2 managers retiring and 4 staying on.  The biggest gap is in the Senior Engineer positions with close to 7 retiring and only 1 left.  But these numbers also don't feel as overwhelming as in the original queries.<br>

|Mentorship Employees |	Retiring Employees |
|---------------------|--------------------|	
|13391	Senior Staff |	28254	Senior Staff|
|13357	Engineer |	14222	Engineer|
|4100	Staff	|	12243	Staff|
|3791	Senior Engineer| 29414	Senior Engineer|
|1946	Assistant Engineer| 1761	Assistant Engineer|
|1812	Technique Leader| 4502	Technique Leader|
|4	Manager |	2 Manager|
