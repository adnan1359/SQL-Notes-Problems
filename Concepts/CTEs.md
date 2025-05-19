
### Common Table Expressions (CTEs) in SQL

  

A **Common Table Expression (CTE)** is a temporary result set in SQL that you can define within a query and reference multiple times in the same query. CTEs are useful for breaking down complex queries into simpler, more readable parts, improving maintainability and clarity. They are defined using the `WITH` clause and exist only for the duration of the query execution.

  

#### Key Characteristics of CTEs

1. **Temporary**: CTEs exist only within the scope of the query in which they are defined.

2. **Named**: You assign a name to the CTE, which you can reference like a table in the main query or subqueries.

3. **Reusable**: A CTE can be referenced multiple times in the same query, unlike subqueries, which are often written repeatedly.

4. **Readable**: CTEs make complex queries easier to understand by modularizing the logic.

5. **Recursive**: CTEs can be recursive, allowing them to reference themselves, which is useful for hierarchical or iterative data processing.

  

#### Syntax of a CTE

```sql

WITH cte_name [(column_name1, column_name2, ...)] AS (

-- Subquery defining the CTE

SELECT ...

FROM ...

WHERE ...

)

-- Main query using the CTE

SELECT ...

FROM cte_name

[WHERE ...]

```

  

- **cte_name**: The name of the CTE, which you use to reference it in the main query.

- **column_name**: Optional; explicitly defines column names for the CTE (if not specified, column names are inferred from the subquery).

- **Subquery**: The query inside the CTE that generates the result set.

- **Main Query**: The outer query that uses the CTE like a table.

  

#### Types of CTEs

1. **Non-Recursive CTEs**: Used for simplifying complex queries by breaking them into manageable parts.

2. **Recursive CTEs**: Used for hierarchical or iterative data, such as organizational charts, bill of materials, or tree structures.

  

---

  

### Code Samples

  

#### 1. Basic Non-Recursive CTE

Suppose you have a table `employees` with columns `employee_id`, `name`, `department_id`, and `salary`. You want to find employees with above-average salaries in their department.

  

```sql

WITH avg_salary_by_dept AS (

SELECT department_id, AVG(salary) AS avg_salary

FROM employees

GROUP BY department_id

)

SELECT e.employee_id, [e.name](http://e.name/), e.salary, e.department_id, a.avg_salary

FROM employees e

JOIN avg_salary_by_dept a

ON e.department_id = a.department_id

WHERE e.salary > a.avg_salary;

```

  

**Explanation**:

- The CTE `avg_salary_by_dept` calculates the average salary per department.

- The main query joins the `employees` table with the CTE to find employees whose salaries exceed their department’s average.

- This breaks down a complex query into a clearer, modular structure.

  

#### 2. Multiple CTEs in a Single Query

You can define multiple CTEs in the same query to handle different parts of the logic.

  

```sql

WITH high_salary_employees AS (

SELECT employee_id, name, department_id, salary

FROM employees

WHERE salary > 50000

),

department_summary AS (

SELECT department_id, COUNT(*) AS emp_count

FROM employees

GROUP BY department_id

)

SELECT [h.name](http://h.name/), h.salary, d.emp_count

FROM high_salary_employees h

JOIN department_summary d

ON h.department_id = d.department_id

WHERE d.emp_count > 5;

```

  

**Explanation**:

- The first CTE (`high_salary_employees`) filters employees with salaries above 50,000.

- The second CTE (`department_summary`) counts employees per department.

- The main query joins the two CTEs to return high-salary employees in departments with more than 5 employees.

  

#### 3. Recursive CTE

Recursive CTEs are used for hierarchical or iterative data. Consider a table `org_chart` with columns `employee_id`, `name`, and `manager_id`, representing an organizational hierarchy.

  

```sql

WITH RECURSIVE employee_hierarchy AS (

-- Anchor member: Start with the top-level employee (e.g., CEO, where manager_id is NULL)

SELECT employee_id, name, manager_id, 1 AS level

FROM org_chart

WHERE manager_id IS NULL

UNION ALL

-- Recursive member: Find employees who report to the previous level

SELECT o.employee_id, [o.name](http://o.name/), o.manager_id, eh.level + 1

FROM org_chart o

JOIN employee_hierarchy eh

ON o.manager_id = eh.employee_id

)

SELECT employee_id, name, manager_id, level

FROM employee_hierarchy

ORDER BY level, name;

```

  

**Explanation**:

- The **anchor member** selects the top-level employee (e.g., the CEO, where `manager_id` is NULL).

- The **recursive member** joins the `org_chart` table with the CTE to find employees who report to the previous level’s employees.

- The `level` column tracks the hierarchy depth.

- The query returns the entire organizational hierarchy, with employees ordered by their level in the hierarchy.

  

#### 4. CTE for Simplifying Aggregations

Suppose you have a `sales` table with columns `order_id`, `region`, `sale_amount`, and `order_date`. You want to calculate the total sales per region and filter regions with total sales above a threshold.

  

```sql

WITH regional_sales AS (

SELECT region, SUM(sale_amount) AS total_sales

FROM sales

GROUP BY region

)

SELECT region, total_sales

FROM regional_sales

WHERE total_sales > 100000

ORDER BY total_sales DESC;

```

  

**Explanation**:

- The CTE `regional_sales` computes the total sales per region.

- The main query filters regions with total sales exceeding 100,000 and sorts them in descending order.

  

---

  

### Benefits of CTEs

- **Improved Readability**: CTEs break down complex queries into logical steps.

- **Reusability**: A CTE can be referenced multiple times in the same query, reducing redundancy.

- **Maintainability**: Easier to modify specific parts of a query without affecting the whole.

- **Recursive Queries**: Enable processing of hierarchical or iterative data, which is difficult with standard SQL.

  

### Limitations of CTEs

- **Temporary Scope**: CTEs exist only for the query they are defined in and cannot be reused across queries.

- **Performance**: In some databases, CTEs may not be optimized as well as subqueries or temporary tables, especially for large datasets.

- **Not Indexable**: Unlike temporary tables, CTEs are not stored physically and cannot be indexed.

  

---

  

### Socratic Questions and Scenario-Based Challenges

  

#### Socratic Questions

1. Why might you choose a CTE over a subquery in a complex query? How does it impact readability and performance?

2. In what scenarios would a recursive CTE be more appropriate than a non-recursive CTE? Can you think of a real-world example where recursion is necessary?

3. How would you decide between using a CTE and a temporary table for a query that needs to reuse a result set multiple times?

4. What are the potential downsides of using multiple CTEs in a single query? How might this affect query execution in a large database?

5. Can you explain how the anchor and recursive members in a recursive CTE work together to produce a result set?

  

#### Scenario-Based Challenges

1. **Sales Performance Analysis**:

- You have a `sales` table with columns `order_id`, `employee_id`, `sale_amount`, and `order_date`. Write a CTE-based query to:

- Identify employees whose total sales in 2024 exceed the average sales of all employees in their department.

- Include a second CTE to calculate the department-level average sales.

- Return the employee’s name, total sales, and department average.

  

2. **Hierarchical Data**:

- Given a table `categories` with columns `category_id`, `category_name`, and `parent_category_id`, write a recursive CTE to display the full hierarchy of categories, including a column that shows the path (e.g., "Electronics > Laptops > Gaming Laptops").

- Ensure the results are ordered by the hierarchy level and category name.

  

3. **Data Cleanup**:

- You have a `transactions` table with columns `transaction_id`, `customer_id`, `amount`, and `transaction_date`. Some customers have multiple transactions on the same date, but you only want the latest transaction per customer per date.

- Use a CTE to identify the latest transaction for each customer-date combination and then calculate the total amount per customer.

  

4. **Complex Aggregation**:

- Using a `products` table with columns `product_id`, `category`, `price`, and `stock_quantity`, write a query with multiple CTEs to:

- Calculate the average price per category.

- Identify products with stock quantities below 10.

- Combine the results to show products with low stock in categories where the average price is above $50.

  

5. **Recursive Challenge**:

- You have a `flights` table with columns `flight_id`, `source_city`, `destination_city`, and `cost`. Write a recursive CTE to find all possible flight paths from a starting city to a destination city, including the total cost and the path (e.g., "New York -> Chicago -> Los Angeles").

- Limit the recursion to a maximum of 3 hops (intermediate cities).
