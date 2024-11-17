# Introduction
:mega: This project focuses on exploring jobs in data analytics. Including revealing top-paying jobs, highly-demanded skills and what skills pays the most. My initial intention was to focus on the New Zealand and Australia market but due to data collection limitation, this project will explore all job postings.

:computer: Check out SQL queries here: [project_sql folder](/project_sql/)

# Background
This project is created under the guidance of Luke Barousse's [SQL Course](https://lukebarousse.com/sql). The goal of this project is to demontrate my understanding of SQL. 

### The questions I wanted to answer through my SQL queries were:

1. What are the top-paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn?

# Tools I used
Key tools used in this project are:

- **SQL**: This is the core of the project. Used to query database and capture crucial insights.
- **PostgreSQL**: One of the most popular database management systems.
- **Visual Studio Code**: Recommended by the course. Ideal for executing SQL queries.
- **Git & GitHub**: Essential platform for sharing SQL scripts and analysis.

# The Analysis
Each query for this project aimed to answer specific question of the data analyst job market. Here's how I approached each question:

### 1. Top Paying Data Analyst Jobs
To find out the highest-paying positions, I filtered out data analyst positions by average annual salary in all locations. This query highlights the high paying jobs on market.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim
ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    -- job_location = 'New Zealand' OR
    -- job_location = 'Australia' AND
    salary_year_avg is NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 50;
```
Here's the breakdown of the top data analyst jobs in 2023:
- **Top Job Titles**: The most frequently listed job titles for data analyst roles include a mix of generic "Data Analyst" roles, as well as specialized positions like "Senior Data Analyst" and "Database Administrator." This suggests that there’s demand for both entry-level and experienced data analysts with specific skill sets.
- **Salary Distribution**: The average annual salary for these roles varies, with a concentration around certain salary bands, suggesting typical pay ranges for data analyst positions.
- **Popular Job Locations**: The data shows the most common locations for these roles, which include major cities or regions with a strong tech and business presence.


![Top Paying Roles](/assets/top_paying_roles.png)
*Bar graph showing the top 50 paid positions. This graph is generated by ChatGPT from my SQL query results*

### 2. Skills for Top Paying Roles
This query finds what skills are required for top-paying positions. Job postings data was joined with skills data to highlight what employers are looking for in high-paid positions.

```sql
WITH top_paying_jobs AS(
    SELECT
    job_id,
    job_title,
    salary_year_avg,
    name AS company_name
FROM
    job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst' AND 
        -- job_location = 'New Zealand' OR
        -- job_location = 'Australia' AND 
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT 
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim
ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim
ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC;

```

Here's the breakdown of the most in demand skills for the top 10 highest paying data analyst jobs in 2023:
- **SQL** is the most mentioned skill leading with 8 occurrences.
- **Python** follows with 7 occurrences.
- **Tableau** follows closely with 6 occurrences.
Other skills including **R** (4 occurrences), **Snowflake** (3 occurrences), **Pandas** (3 occurrences) and **Excel** (3 occurrences) are also highly preferred.

![Top Paying Skills](/assets/top_paying_skills.png)
*Bar graph showing the top 10 highest paid skilled for data analysts. his graph is generated by ChatGPT from my SQL query results*

### 3. In-Demand Skills for Data Analysts

This query finds the most frequently requested skills in job postings.

```sql
SELECT 
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' 
GROUP BY
    skills
ORDER BY
    demand_count DESC
LIMIT 5;
```
Here's the breakdown of the most demanded skills for data analysts in 2023:

- **SQL** and **Excel** remain core tools on market. SQL remains a fundamental skill, essential for querying and managing large datasets, while Excel is indispensable for data manipulation and quick analyses. These tools are foundational and continue to be widely used across various levels of data work.
- The integration of **Programming** and **Visualization Tools** into the data analyst skill set reflects the shift towards a more technical and narrative-driven approach to data.

| Skills   | Demand Count |
|----------|--------------| 
| SQL      | 92628        |
| Excel    | 67031        |
| Python   | 57326        |
| Tableau  | 46554        |
| Power BI | 39468        |

*Table of the top 5 demanded skills in data analyst job posting*

### 4. Skills Based on Salary
Finding out which skills offer the highest pay.
```sql
SELECT 
    skills,
    ROUND(AVG(salary_year_avg),0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    salary_year_avg is NOT NULL AND
    job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```
Here's a breakdown of the results for top paying skills for data analysts:

- The data analyst industry greatly values **Big Data Processing** (PySpark, Couchbase), **Machine Learning** (DataRobot, Jupyter), **Aritificial Intellegence** (Watson), and **Python** libraries (Pandas).
- Knowledge in **Software Development** (Gitlab, Swift) shows that engineering crosses path with data analysis and is highly remunerative.

| Skills        | Average Salary ($)|
|---------------|-------------------|
| pyspark       |            208,172|
| bitbucket     |            189,155|
| couchbase     |            160,515|
| watson        |            160,515|
| datarobot     |            155,486|
| gitlab        |            154,500|
| swift         |            153,750|
| jupyter       |            152,777|
| pandas        |            151,821|
| elasticsearch |            145,000|

*Table of average salary for the top 10 paying skills for data analysts*

### 5. Most Optimal Skills to Learn

This query combined demand and salary data to find out what skills are both in demand and offers high salary.

```sql
WITH skills_demand AS(
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' AND
        salary_year_avg is NOT NULL AND
        job_work_from_home = True
    GROUP BY
        skills_dim.skill_id
), avgerage_salary AS(
    SELECT
        skills_job_dim.skill_id,
        ROUND(AVG(job_postings_fact.salary_year_avg),0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' AND
        salary_year_avg is NOT NULL AND
        job_work_from_home = True
    GROUP BY
        skills_job_dim.skill_id
)

SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM
    skills_demand
INNER JOIN avgerage_salary
ON skills_demand.skill_id = avgerage_salary.skill_id
WHERE
    demand_count > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```

| Skills     | Demand Count | Average Salary ($) |
|------------|--------------|--------------------|
| go         |            27|             115,320|
| confluence |            11|             114,210|
| hadoop     |            22|             113,193|
| snowflake  |            37|             112,948|
| azure      |            34|             111.225|
| bigquery   |            13|             109,654|
| aws        |            32|             108,317|
| java       |            17|             106,906|
| ssis       |            12|             106,683|
| jira       |            20|             104,918|

*Table of the most optimal skills for data analyst sort by salary*

| Skills     | Demand Count | Average Salary ($) |
|------------|--------------|--------------------|
| sql        |           398|              97,237|
| excel      |           256|              87,288|
| python     |           236|             101,397|
| tableau    |           230|              99,288|
| r          |           148|             100,499|
| power bi   |           110|              97,431|
| sas        |            63|              98,902|
| powerpoint |            58|              88,701|
| looker     |            49|             103,795|
| word       |            48|              82,576|

*Table of the most optimal skills for data analyst sort by demand count*

Here's a breakdown of the most optimal skills for data analysts in 2023:
- **Cloud Tools and Technologies** (Snowflake, Azure, AWS and BigQuery) are the dominant skills that pays the most in data analyst.
- **Programming Languages** (Python, R) are in high demand and highly favored by the industry.
- Knowledge for **Business Intelligence and Visualization Tools** (Tableau, Power BI, Looker) are essential for data analysts.

# What I Learned

Throughout this project, I have learn SQL in a systemic manner. Here are some of my key take-aways:

- :crystal_ball: **Mastering Advanced Query Crafting:** mastered the skill of joining tables, subquries and CTEs.
- :hammer: **Github and VS Manoeuvre:** familiered myself with the connetion of Git, Github, Visual Studio Code and PostgreSQL
- :triangular_ruler: **Interpreting Data for Insights:** telling a story from the results is the next step after writing correct queries. Data visualization is also a crucial step.

# Conclusions

### Insights
From the analysis, several general insights emerged:

1. **Top-Paying Data Analyst Jobs**: highest-paying data analyst job has a wide range of pay of pay scale. The highest job listing can be as high as 
2. **Top-Paying Job Skills**: High-paying jobs require skills include SQL, Python and Tableau.
3. **Most In-Demand Skills**: SQL is the most demanded skill in the data analyst job market.
4. **Skills That Pays High**: Specialized skills in big data processing and machine learning has the highest average salary.
5. **Optimal Skills for Job Market Value**: SQL is in high demand and offers a high average salary. Programming languages (Python and R) are also preferred by the market.

### Closing Thoughts

I have build a sound SQL foundation through this project. This project also gave me some market insights in the data analyst industry. I have learnt that mastering SQL is essential to obtain a role. I also understood the market demand and helped me understand how to increase competitivesness and salary potential. This project also serves as a guide for future learning. I understood that programming, data visualization and cloud data platforms are currently most sought-after. Overall, data analytics is an forever evolving field. Adapting to new tools and technologies is key to stay relevant. Embracing a mindset of continuous learning will help me stay aligned with industry trends and keeping my skills fresh.

