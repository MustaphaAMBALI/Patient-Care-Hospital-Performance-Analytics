# Patient-Care-Hospital-Performance-Analytics
A deep dive into Healthcare Analytics using SQL.

## Project Objective
To analyze hospital admissions, treatment costs, and patient behavior using SQL.  
The goal is to provide actionable insights for hospital management regarding costs, length of stay, and patient demographics.

## Dataset Overview
The project uses a synthetic healthcare dataset consisting of 4 tables:
- **Patients**: demographic info (age, gender, location, insurance type).
- **Admissions**: admission type, admission date, length of stay, discharge status.
- **Billing**: treatment costs linked to admissions.
- **Satisfaction**: satisfaction score, loyalty level and support contacted linked to admissions.

## Methodology
- **Data Modeling**: Created relationships across Patients, Admissions, and Billing.
- **Data Cleaning**: Handled NULLs in cost and normalized admission types.
- **Queries**:

- Which departments have the highest admissions and readmission rates?
```
SELECT Department,
		COUNT(Admission_ID) AS [Total Admissions],
		SUM(CASE WHEN Readmitted = 'True' THEN 1 ELSE 0 END) AS Readmissions,
		ROUND(100.0 * SUM(CASE WHEN Readmitted = 'True' 
			THEN 1 ELSE 0 END) / COUNT(Admission_ID),2) AS [Readmission Rates]
FROM Admissions
GROUP BY Department
ORDER BY [Total Admissions] DESC;
```

- Do emergency cases result in higher costs or longer stays compared to routine admissions?
```
 SELECT A.Admission_Type,
	ROUND(AVG(A.Length_of_Stay),2) AS [Avg Duration],
	ROUND(AVG(B.Treatment_Cost),2) AS [Avg Cost]
FROM Admissions A
LEFT JOIN Billing B
	ON A.Admission_ID = B.Billing_ID
GROUP BY Admission_Type;
```

- What age groups and locations contribute most to hospital admissions?
```
SELECT
	CASE 
		WHEN P.Age < 18 THEN '0-17'
		WHEN P.Age BETWEEN 18 AND 35 THEN '18-35'
		WHEN P.Age BETWEEN 36 AND 55 THEN '36-55'
		WHEN P.Age BETWEEN 56 AND 75 THEN '56-75'
		ELSE '76+' 
	END AS Age_Groups,
	P.Location,
	COUNT(A.Admission_ID) AS Total_Admission
FROM Patients P
JOIN Admissions A
	ON A.Patient_ID = P.Patient_ID
GROUP BY 
	CASE 
		WHEN P.Age < 18 THEN '0-17'
		WHEN P.Age BETWEEN 18 AND 35 THEN '18-35'
		WHEN P.Age BETWEEN 36 AND 55 THEN '36-55'
		WHEN P.Age BETWEEN 56 AND 75 THEN '56-75'
		ELSE '76+' 
	END,
	P.Location
ORDER BY Total_Admission DESC;
```

- How do treatment costs vary by department and admission type?
```
SELECT
	A.Department,
	A.Admission_Type,
	ROUND(AVG(B.Treatment_Cost),2) AS [Avg Treatment Cost],
	MIN(B.Treatment_Cost) AS Min_Cost,
	MAX(B.Treatment_Cost) AS Max_Cost
FROM Admissions A
JOIN Billing B
	ON A.Admission_ID = B.Admission_ID
GROUP BY A.Department, A.Admission_Type
ORDER BY A.Department, A.Admission_Type;
```

- What factors drive higher satisfaction scores (support contact, cost, length of stay)?
```
SELECT
	A.Department,
	ROUND(AVG(S.Satisfaction_Score),2) AS Avg_Score,
	ROUND(AVG(A.Length_of_Stay),2) AS Avg_Duration,
	ROUND(AVG(B.Treatment_Cost),2) AS Avg_Cost,
	SUM(CASE 
			WHEN S.Support_Contacted = 'True' THEN 1 ELSE 0
		END) AS Support_Cases
FROM Admissions A
JOIN Satisfaction S ON S.Admission_ID = A.Admission_ID
JOIN Billing B ON A.Admission_ID = B.Admission_ID
GROUP BY A.Department
ORDER BY Avg_Score DESC;
```

- Is there a correlation between satisfaction level and loyalty?
```
SELECT
	Loyalty_Level,
	ROUND(AVG(Satisfaction_Score),2) AS Avg_Score,
	COUNT(Satisfaction_ID) AS Total_Satisfaction
FROM Satisfaction
GROUP BY Loyalty_Level
ORDER BY Avg_Score DESC;
```

## Tools
- Microsoft SQL Server
- T-SQL for queries
- GitHub for version control

## Key Insights
- Emergency admissions average higher costs and longer stays than elective admissions.
- Insured patients tend to have shorter average stays than uninsured ones.
- Readmissions within 30 days are highest among elderly patients .
- Treatment costs vary significantly by admission type and insurance.

## Recommendations
- Hospitals should strengthen preventive care programs for elderly patients.
- Consider reviewing emergency care protocols to optimize costs and length of stay.
- Tailor insurance packages to reduce repeat admissions.

---

