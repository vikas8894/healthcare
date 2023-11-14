# healthcare
# analysed data using sqlite

SELECT *
  FROM healthcare;
  
  
-- finding out the total number of patients in respect to gender

SELECT gender,
       count(gender) 
  FROM healthcare
 GROUP BY 1;
 -- out of 10000 patients 5075 are Female and 4925 are Male
 

-- finding out the the number of different blood group people and percentage in respect to total population

WITH table1 AS (
    SELECT gender,
           blood_type,
           count(blood_type) AS blood_group_count
      FROM healthcare
     GROUP BY 1,
              2
),
table2 AS (
    SELECT count(gender) AS q
      FROM healthcare
)
SELECT gender,
       blood_type,
       blood_group_count,
       round(blood_group_count * 100.0 / q, 2) AS blood_group_percentage
  FROM table1 AS t1
       JOIN
       table2 AS t2
 ORDER BY 1;
 

 -- finding number of people in different age group having what medical conditons:

SELECT CASE WHEN age > 20 AND 
                 age < 35 THEN '20-35' WHEN age > 34 AND 
                                            age < 50 THEN '35-50' WHEN age > 49 AND 
                                                                       age < 65 THEN '50-65' WHEN age > 64 AND 
                                                                                                  age < 80 THEN '65-80' ELSE 'above 80' END AS age_group,
       medical_condition,
       count(medical_condition) AS no_of_people_having_different_medical_condition
  FROM healthcare
 GROUP BY 1,
          2;


-- finding which age group has maximum medical condition:

WITH table3 AS (
    SELECT CASE WHEN age > 20 AND 
                     age < 35 THEN '20-35' WHEN age > 34 AND 
                                                age < 50 THEN '35-50' WHEN age > 49 AND 
                                                                           age < 65 THEN '50-65' WHEN age > 64 AND 
                                                                                                      age < 80 THEN '65-80' ELSE 'above 80' END AS age_group,
           medical_condition,
           count(medical_condition) AS no_of_people_having_different_medical_condition
      FROM healthcare
     GROUP BY 1,
              2
)
SELECT age_group,
       medical_condition,
       sum(no_of_people_having_different_medical_condition) OVER (PARTITION BY age_group) AS sum_of_medical_condition_with_respect_age_group
  FROM table3;
  -- it is found that people in age group of 65-80 has maximum medical condition and people with age of above 80 has least medical_conditon as per reports butit may be assumed that there
-- less data of above 80 age group people then it is found people in age group of 20-35 has least medical condition.


-- finding in every age group which medical condition is common:

WITH table3 AS (
    SELECT CASE WHEN age > 20 AND 
                     age < 35 THEN '20-35' WHEN age > 34 AND 
                                                age < 50 THEN '35-50' WHEN age > 49 AND 
                                                                           age < 65 THEN '50-65' WHEN age > 64 AND 
                                                                                                      age < 80 THEN '65-80' ELSE 'above 80' END AS age_group,
           medical_condition,
           count(medical_condition) AS no_of_people_having_different_medical_condition
      FROM healthcare
     GROUP BY 1,
              2
)
SELECT age_group,
       medical_condition,
       max(no_of_people_having_different_medical_condition) AS sum_of_medical_condition_with_respect_age_group
  FROM table3
 GROUP BY 1;
 -- It is found that cancer is most common in age group 35-50 and 50-65 whereas hypertension is moslty seen in age group of 35-50 and above 80 whereas in age group of 20-35 Asthma is much common.
-- adding new column in table age_group


-- finding what age group chosses what insurance companies

SELECT age_group,
       insurance_provider,
       count(insurance_provider) 
  FROM healthcare
 GROUP BY 1,
          2;
          

-- which insurance provider is maximum selected as per different age group

WITH table4 AS (
    SELECT age_group,
           insurance_provider,
           count(insurance_provider) AS n
      FROM healthcare
     GROUP BY 1,
              2
)
SELECT age_group,
       insurance_provider,
       max(n) AS maximum_selection
  FROM table4
 GROUP BY 1;
 -- It is observed that Medicare is mostly selected by age group of 35-50. Cigna is mostly selected by age group of 20-35 and 50-65, UnitedHealthCare is mostly selected by age group of above 80,
-- and Aetna is mostly opted by age group between 65-80.However, cigna in total is mostly selected insurance_provider


-- which medical condition has maximum treatment cost in total:

SELECT sum(billing_amount),
       medical_condition
  FROM healthcare
 GROUP BY 2
 ORDER BY 1;
 -- it is observed that Arthritis has maximum treatment cost followed by obesity while Cancer has least treatment cost in total compared to other 6 medical conditions.
 

-- which hospital has maximum billing amount and for what medical condition:

SELECT hospital,
       medical_condition,
       max(billing_amount) 
  FROM healthcare
 GROUP BY 1,
          2
 LIMIT 1;
 -- Abbot Inc has maximum billing amount for Arthritis.
 

-- which insurance provider helped customer in getting least treatment cost

SELECT insurance_provider,
       medical_condition,
       min(billing_amount) 
  FROM healthcare
 GROUP BY 1,
          2
 LIMIT 1;
 -- Aetna is insurance provider that got min billing amount for the customer
 

-- finding what admission type is there in different medical condition:

SELECT medical_condition,
       Admission_Type,
       count(Admission_Type) 
  FROM healthcare
 GROUP BY 1,
          2;
          

-- finding what is maximum admission type in different medical condition:

WITH table5 AS (
    SELECT medical_condition,
           Admission_Type,
           count(Admission_Type) AS count
      FROM healthcare
     GROUP BY 1,
              2
)
SELECT medical_condition,
       admission_type,
       max(count) AS maximum_patients_admitted_type
  FROM table5
 GROUP BY 1;
 -- it is observed in case of cancer, diabetes and Obesity patients are maximum admitted in emergency situation while in case of Asthma and Hypertension patients are admitted under Urgent condition and
-- in case of Arthritis patients are admitted under elective type.


-- what kind of medication is given in different medical_condition:

SELECT medical_condition,
       medication
  FROM healthcare
 GROUP BY 1;
 

-- finding the count of different results when given different medication on different medical conditions

SELECT medical_condition,
       medication,
       Test_Results,
       count(test_results) 
  FROM healthcare
 GROUP BY 1,
          2,
          3
 ORDER BY 1;
 

-- which blood group is prone to what kind of condition

WITH table6 AS (
    SELECT blood_type,
           medical_condition,
           count(medical_condition) AS r
      FROM healthcare
     GROUP BY 1,
              2
)
SELECT blood_type,
       medical_condition,
       max(r) AS count_of_different_medical_condition
  FROM table6
 GROUP BY 1;
 -- it is found that patients with blood group A+,B+ and O+ suffers with more of Asthma, whereas pateints with blood group of A- and B- suffers with Cancer,
-- patients withs blood group AB+ and O- suffers with Arthritis and patients with AB- suffers maximum of Obesity.


-- find maximum days taken to get discharge as categorised by medical_condition

WITH table7 AS (
    SELECT medical_condition,
           julianday(Discharge_Date) - julianday(date_of_admission) AS n
      FROM healthcare
     GROUP BY 1
)
SELECT medical_condition,
       max(n) AS days_taken_to_discharge
  FROM table7
 GROUP BY 1;
 -- It is found that patientwith Obesity took maximum of 30 days to get discharge,patient with cancer took maximum of 28 days to get discharge , patient with Hypertension took
-- 24 days whereas patient with Arthritis,Asthma and Diabetes took 14 days to get discharged.


-- find maximum days taken to get discharge as categorised by medical condition when test results are normal:

WITH table7 AS (
    SELECT medical_condition,
           julianday(Discharge_Date) - julianday(date_of_admission) AS n
      FROM healthcare
     WHERE test_results LIKE 'N%'
     GROUP BY 1
)
SELECT medical_condition,
       max(n) AS days_taken_to_discharge
  FROM table7
 GROUP BY 1;
