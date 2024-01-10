# Who Is Leaving Laxana Development?
![project1image](https://github.com/jmwaigom/Employee-Atttrition-Analytics/assets/155841258/7aede357-8ae5-4074-afc8-b922d4d0078c)

## Employee Attrition Analytics - SQL Project

### Table of Contents
- [Project Overview](#project-overview)
- [Objectives](#objectives)
- [Assumptions](#assumptions)
- [Data Source](#data-source)
- [Data Analysis](#data-analysis)
- [Results](#results)
- [Recommendations](#recommedations)
- [Limitations](#limitations)

### Project Overview

The board of directors at Laxana Development just appointed a new Chief Executive Office (CEO) to oversee company operations moving forward. 
One of the major objectives of the new CEO within the first 90 days in office is to design an initiative that will encourage employee retention within the company. 
He believes that the company employs great talent, so he wants to keep attrition rate as low as 5% across the company and 10% in each of the three departments (Sales, Human Resources/HR and Research & Development/RnD). 
Before he sets any plan in motion, he wants to understand the current state of employee attrition. So he has cascaded an assignment down to a junior analyst to conduct a preliminary exploaration to find out the current 
rate of attrition and the most affected department and roles. Finally, he wants recommendations on further steps

### Objectives
1. Calculate current attrition rate across the company
2. Examine departmental attrition rate as percentage of total employees per department
3. For each department, examine the average tenure of attrited employees versus current ones
4. Examine attrition rate according to job role and department. Identify three top roles with leading attrition rates
5. For the three top roles with leading attrition rates, compare average tenure for both attrited and current employees

### Assumptions
Employee attrition could be due to voluntary resignation, termination or death. However, in this analysis, attrition has been fully attributed to voluntary resignation for two reasons. First, there is no data to reflect death or termination within the dataset. Second, this approach allows the company to focus more on enhancing employee's experience 

### Data Source
The primary dataset used for this analysis was 'WA_Fn-UseC_-HR-Employee-Attrition.csv' sourced from Kaggle. The dataset was downloaded and renamed to 'hr_data.csv', which was then imported to MySQL for analysis. The dataset was sourced from Kaggle, and it can be downloaded [here](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset/data)

### Data Analysis

#### Task 1: Calculate the attrition rate across the company 
 Using subquery in the SELECT clause to calculate total employee count. Calculating attrited employees in the main query then dividing the number by the total employee count in the subquery. multiplying by 100 to get a percentage, then rounding to 1 decimal place
```
SELECT
	ROUND((COUNT(EmployeeNumber)/
    (SELECT 
	COUNT(EmployeeNumber) 
	FROM hr_data)) * 100,1) AS CompanyAttritionRate
FROM hr_data
WHERE Attrition = 'Yes'
```
From the query above, the company attrition rate was found to be 16.1%

#### Task 2: Calculate the departmental attrition rate as a percentage of total employees per department

Creating a view to display a subset of data with columns relevant to calculate
rate of attrition by department. This view will only have the following columns
EmployeeNumber, Department, Attrition, Age and EmployeeCount
```
CREATE VIEW emp_table AS
SELECT EmployeeNumber, Department, Attrition, Age, EmployeeCount, YearsAtCompany
FROM hr_data;
```
Using CTE to  group employees by department and add a new column which calculates the number of attrited employees. Afterwards, Calculating attrition rate by dividing number of attrited employees 
by total number of employees in a department
```
WITH DepartmentAttrition AS 
	(SELECT 
		Department,
		SUM(EmployeeCount) AS TotalEmployees,
		SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END) AS AttritedEmployees
	FROM emp_table
	GROUP BY Department)  

SELECT 
	*,
    ROUND(((AttritedEmployees/TotalEmployees) * 100),1) AS AttritionRate
FROM DepartmentAttrition
ORDER BY AttritionRate DESC;

```
![Task2](https://github.com/jmwaigom/Employee-Atttrition-Analytics/assets/155841258/5a4ff8e5-6c23-44c3-8250-7e011e911b83)

#### Task 3: For each department, what is the average tenure of an attrited employee versus a current one?
Using aggregate functions to calculate average tenure for attrited and current employees
```
SELECT 
	Department,
    ROUND(AVG(CASE WHEN Attrition = 'Yes' THEN YearsAtCompany ELSE NULL END)) AS AverageTenureAttrited,
	ROUND(AVG(CASE WHEN Attrition = 'No' THEN YearsAtCompany ELSE NULL END)) AS AverageTenureCurrent
FROM hr_data
GROUP BY Department
ORDER BY AverageTenureAttrited;  

```
![Task3](https://github.com/jmwaigom/Employee-Atttrition-Analytics/assets/155841258/a58ff0b9-dc69-4c56-82f6-6dc711f9a12c)


#### Task 4: Display the number of attritted employees according to job role and department
Creating CTE to display total number of attrited employees according to job role and department. Afterwards, using window functions to calculate percentage (attrition rate) according to job role and department
```
WITH AttritionByRole AS 
	(SELECT 
		JobRole,
		Department,
		COUNT(EmployeeNumber) AS AttritedEmployees
	FROM hr_data
	WHERE Attrition = 'Yes'
	GROUP BY JobRole, Department
	ORDER BY AttritedEmployees DESC)

SELECT 
	*,
    ROUND((AttritedEmployees * 100 / SUM(AttritedEmployees) OVER (PARTITION BY Department)),1) AS DepartmentAttritionRate,
    ROUND((AttritedEmployees * 100 / SUM(AttritedEmployees) OVER ()),1) AS OverallAttritionRate
FROM AttritionByRole
ORDER BY OverallAttritionRate DESC
-- LIMIT 3;

```
![Task4](https://github.com/jmwaigom/Employee-Atttrition-Analytics/assets/155841258/c2ddac67-34cd-485e-a4f4-82a3c0ab5524)

![Task4b](https://github.com/jmwaigom/Employee-Atttrition-Analytics/assets/155841258/3206d764-32cc-4867-b81b-f0472c462ebe)


#### Task 5: Based on results from task 4, it appears that Laboratory Technician, Sales Executive and Research Scientist roles experienced the highest attrition rates. Compare the average tenure of these roles for both attrited and current employees
```
SELECT 
    JobRole,
    ROUND(AVG(CASE WHEN Attrition = 'Yes' THEN YearsAtCompany ELSE NULL END), 1) AS AttEmpAvgTenure,
    ROUND(AVG(CASE WHEN Attrition = 'No' THEN YearsAtCompany ELSE NULL END), 1) AS CurrEmpAvgTenure
FROM hr_data
WHERE JobRole IN ('Laboratory Technician', 'Sales Executive', 'Research Scientist')
GROUP BY JobRole;

```
![Task55](https://github.com/jmwaigom/Employee-Atttrition-Analytics/assets/155841258/b758fe86-0e75-45d2-9a76-ca1dc1f90af9)

### Results
After the analysis, the attrition rate was found to be 16.1% which is about three times higher than the target (5%). Sales has the highest rate of 20.6%, followed by HR (19.0)%, then RnD (13.8%).   

For attrited employees, HR had the shortest tenure of 4 years on average. Sales had the highest tenure of 6 years followed by RnD, 5 years. For current employees, HR and Sales averaged 8 years while RnD averaged 7 years. Generally, it appears that on average, majority of attrition tends to happen between the 4th and 8th year with the company.

The leading role in attrition was Laboratory Technician at 26.2%, followed by Sales Executive at 24.1%, then Research Scientist at 19.1% (These are percentages of overall attrited employees across all three departments. The trend is consistent within departments as well).
On average, attrited Laboratory Technicians stayed the shortest (about 3 years) while Sales Executives stayed the longest (nearly 7 years). However, the average tenure of a current Sales Executive is nearly 8 years. This indicates a likelihood of impending attrition in the Sales Executive Role.

In general, Sales Executive role is anticipated to see more attrition in the near future despite having longer average tenure. Laboratory Technicians stay the shortest and experience higher attrition rate. 

### Recommendations

This was a preliminary analysis conducted to identify the painpoints. The next recommended step would be to conduct a diagnostic analysis. The aim of this analysis would be to identify the potential cause of elevated attrition in the departments and roles identified in this analysis. For example, since it has been established that Laboratory Technicians attrite at the highest rate and stay the shortest compared to other roles,  the next step would be to dig into factors such as compesation, work-life balance and/or job satisfaction, etc.

### Limitations
This analysis relied heavily on average values, which could skew results in case of outliers or significant disproportionate distribution of values.











