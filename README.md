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
Provide a bulleted list with four major points from the two analysis deliverables. Use images as support where needed.

## Summary
Provide high-level responses to the following questions, then provide two additional queries or tables that may provide more insight into the upcoming "silver tsunami."

How many roles will need to be filled as the "silver tsunami" begins to make an impact?

Are there enough qualified, retirement-ready employees in the departments to mentor the next generation of Pewlett Hackard employees?
