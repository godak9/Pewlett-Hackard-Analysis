# Pewlett-Hackard Employee Database Analysis with SQL
## Overview 
Managers must be prepared for the tens of thousands of current employees preparing to retire and what some call the "silver tsunami". Pewlett-Hackard is a company that was slightly under-prepared to determine the amount of employees retiring and the amount of positions that need to be filled becasue they lacked an employee database that available employee data easily accessible. The task for this project was to upload the six CSV files of employee data Pewlett-Hackard had been using into tables in one database using SQL where these files could be connected and referenced easily to extract the data needed to determine the number of retiring employees per title and to identify employees who are eligible to participate in a mentorship program to fill their vacancies.

The six CSV files used for the analysis can be found in the "Data" folder (departments.csv, dept_emp.csv, dept_managers.csv, employees.csv, salaries.csv, titles.csv). The files were imported into tables in SQL, and an ERD was created to show the relationships between the tables:
![EmployeeDB](https://user-images.githubusercontent.com/104794100/181667803-1a8716c4-8363-4d01-9f65-12df60e51ee4.png)
The code used to create these tables in SQL can be found in the schema.sql file and also shown below:
```
-- Creating tables for PH-EmployeeDB
CREATE TABLE departments (
     dept_no VARCHAR(4) NOT NULL,
     dept_name VARCHAR(40) NOT NULL,
PRIMARY KEY (dept_no),
UNIQUE (dept_name)
);

CREATE TABLE employees (
	   emp_no INT NOT NULL,
     birth_date DATE NOT NULL,
     first_name VARCHAR NOT NULL,
     last_name VARCHAR NOT NULL,
     gender VARCHAR NOT NULL,
     hire_date DATE NOT NULL,
PRIMARY KEY (emp_no)
);

CREATE TABLE dept_manager (
	  dept_no VARCHAR(4) NOT NULL,
    emp_no INT NOT NULL,
    from_date DATE NOT NULL,
    to_date DATE NOT NULL,
FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
PRIMARY KEY (emp_no, dept_no)
);

CREATE TABLE salaries (
	  emp_no INT NOT NULL,
	  salary INT NOT NULL,
	  from_date DATE NOT NULL,
	  to_date DATE NOT NULL,
FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
PRIMARY KEY (emp_no)
);

CREATE TABLE titles (
	  emp_no INT NOT NULL,
	  title VARCHAR (40) NOT NULL,
	  from_date DATE NOT NULL,
	  to_date DATE NOT NULL,
FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
);

CREATE TABLE dept_emp (
	  dept_no VARCHAR (4) NOT NULL,
	  emp_no INT NOT NULL,
	  from_date DATE NOT NULL,
	  to_date DATE NOT NULL,
FOREIGN KEY (dept_no) REFERENCES departments (dept_no),
FOREIGN KEY (emp_no) REFERENCES employees (emp_no),
PRIMARY KEY (dept_no,emp_no)
);
```

## Results

- Retrieving data on each employee about to retire by title 

It was determined that anyone born between 1952 and 1955. The manager wanted to know the title of every retiree. The follwoing code was used to create a new table that contained title information (also found in the Employee_Database_Challenge.sql file in the "Queries" folder):
```
SELECT e.emp_no, 
    e.first_name, 
    e.last_name,
	  t.title, 
    t.from_date, 
    t.to_date
INTO retirement_titles
FROM employees as e
JOIN titles as t
ON e.emp_no = t.emp_no
WHERE e.birth_date BETWEEN '1952-01-01' AND '1955-12-31'
ORDER BY e.emp_no ASC;
```
This table was exported and saved as a CSV file which can found in the "Data" folder in the retirement_titles.csv file.

- Retrieving the most recent title for each employee about to retire 

There were duplicate entries for some employees because they have switched titles over the years. The DISTINCT ON statement was used to retrieve the first occurrence of the employee number for each set of rows defined by the ON () clause. The following code was used to remove duplicates and keep only the most recent title of each *current* employee eligible for retirement (also found in the Employee_Database_Challenge.sql file in the "Queries" folder):
```
SELECT DISTINCT ON (rt.emp_no) 
    rt.emp_no,
	rt.first_name,
	rt.last_name,
	rt.title
INTO unique_titles 
FROM retirement_titles AS rt 
WHERE (rt.to_date='9999-01-01')
ORDER BY emp_no ASC, to_date DESC;
```
This table was exported and saved as a CSV file which can found in the "Data" folder in the unique_titles.csv file.

- Retrieving the number of employees by their most recent job title who are about to retire

Employees about to retire were grouped by title and counted. The following code was used to retrieve these counts (also found in the Employee_Database_Challenge.sql file in the "Queries" folder):
```
SELECT COUNT(title),
	u.title
INTO retiring_titles
FROM unique_titles as u
GROUP BY u.title
ORDER BY count DESC;
```
The following is an image of table which was also saved and exported as a CSV file thatcan found in the "Data" folder in the retiring_titles.csv file: 

![Screen Shot 2022-07-28 at 10 57 17 PM](https://user-images.githubusercontent.com/104794100/181673544-032d7e8c-8d42-437f-b944-6967db19a5a4.png)

- Retrieving the employees eligible to participate in a mentorship program

Current employees born in 1965 are eligible to participate in a mentorship program to help fill roles of retiring employees. The following code was used to retrieve the data on these eligible employees (also found in the Employee_Database_Challenge.sql file in the "Queries" folder):
```
SELECT DISTINCT ON (e.emp_no)
	e.emp_no,
	e.first_name, 
	e.last_name, 
	e.birth_date,
	de.from_date,
	de.to_date,
	t.title
INTO mentorship_eligibilty
FROM employees as e
INNER JOIN dep_emp as de
ON (e.emp_no = de.emp_no)
INNER JOIN titles as t
ON (e.emp_no = t.emp_no)
WHERE (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31')
AND (de.to_date = '9999-01-01')
ORDER BY e.emp_no ASC;
```
This table was exported and saved as a CSV file which can found in the "Data" folder in the mentorship_eligibility.csv file.

## Summary
Based on the analysis is was determined that 72,458 employees are eligible for retirement. However, there are only 1,549 employees eligible to participate in the mentorship program to help fill these postitions. Pewlett-Hackard should consider expanding the eligibilty requirements for participation in the program to account for the massive difference in amount of positions need to be filled compared to the amount of employees able to mentor the next generation to fill these postitions. 
If eligibility requirments were expanded to also include employees born in 1966 and 1967, more mentors could be avaible to provide training. To find employees born in 1965, 1966, and 1967 a new query should be generated using the following updated code:
```
SELECT DISTINCT ON (e.emp_no)
	e.emp_no,
	e.first_name, 
	e.last_name, 
	e.birth_date,
	de.from_date,
	de.to_date,
	t.title
INTO mentorship_eligibilty
FROM employees as e
INNER JOIN dep_emp as de
ON (e.emp_no = de.emp_no)
INNER JOIN titles as t
ON (e.emp_no = t.emp_no)
WHERE (e.birth_date BETWEEN '1965-01-01' AND '1967-12-31')
AND (de.to_date = '9999-01-01')
ORDER BY e.emp_no ASC;
```
