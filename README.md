# online_music_store_data_analysis.
Here’s a **professional and well-structured `README.md` file** for your GitHub repository on the **SQL project analyzing online music store data**.
It includes an introduction, dataset overview, objectives, ERD reference, questions with categorized levels, and technical details.
You can copy-paste this into a `README.md` file in your GitHub project.

---

# 🎵 SQL Project: Online Music Store Data Analysis

## 📖 Overview

This project explores and analyzes an **online music store database** using **SQL** to derive business insights.
The objective is to answer a series of real-world analytical questions — from identifying top customers to finding the most popular genres — using SQL queries of varying complexity (Easy, Moderate, and Advanced levels).

This project demonstrates my ability to:

* Query normalized relational databases.
* Use **JOINs, CTEs, Recursive CTEs, and Window Functions**.
* Solve **business-oriented analytical problems** with SQL.
* Present clean and efficient solutions to real data scenarios.

---

## 🗂️ Dataset Information

The database simulates an online music store (similar to Chinook DB) and includes the following key tables:

| Table            | Description                                                             |
| ---------------- | ----------------------------------------------------------------------- |
| **employee**     | Details about employees, their roles, and hierarchy levels.             |
| **customer**     | Information about customers, including contact and billing data.        |
| **invoice**      | Contains customer purchases, billing country, and total amounts.        |
| **invoice_line** | Line-level details of each invoice (tracks purchased, price, quantity). |
| **track**        | Contains all track details, such as name, album, genre, and duration.   |
| **album**        | Details about music albums, including associated artists.               |
| **artist**       | Information about artists and bands.                                    |
| **genre**        | Contains music genre data such as Rock, Jazz, Pop, etc.                 |

---

## 🧩 Entity Relationship Diagram (ERD)

The database relationships are illustrated below:

```
Customer --< Invoice --< InvoiceLine >-- Track >-- Album >-- Artist
                                              |
                                              v
                                            Genre
```

* One **Customer** can have multiple **Invoices**
* Each **Invoice** contains multiple **Invoice Lines**
* Each **Invoice Line** refers to a **Track**
* Each **Track** belongs to an **Album** and a **Genre**
* Each **Album** belongs to one **Artist**

---

## 🎯 Project Objectives

This project is divided into three question sets based on difficulty:

1. **Question Set 1 – Easy:** Basic queries, grouping, and sorting
2. **Question Set 2 – Moderate:** Multi-table joins, subqueries, and filtering
3. **Question Set 3 – Advanced:** Common Table Expressions (CTEs), Recursive queries, and Window Functions

---

## 🧠 Question Set 1 – Easy

### Q1. Who is the senior most employee based on job title?

```sql
SELECT title, last_name, first_name 
FROM employee
ORDER BY levels DESC
LIMIT 1;
```

### Q2. Which countries have the most invoices?

```sql
SELECT COUNT(*) AS c, billing_country 
FROM invoice
GROUP BY billing_country
ORDER BY c DESC;
```

### Q3. What are the top 3 values of total invoice?

```sql
SELECT total 
FROM invoice
ORDER BY total DESC
LIMIT 3;
```

### Q4. Which city has the best customers (highest revenue)?

```sql
SELECT billing_city, SUM(total) AS InvoiceTotal
FROM invoice
GROUP BY billing_city
ORDER BY InvoiceTotal DESC
LIMIT 1;
```

### Q5. Who is the best customer (highest spender)?

```sql
SELECT customer.customer_id, first_name, last_name, SUM(total) AS total_spending
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total_spending DESC
LIMIT 1;
```

---

## ⚙️ Question Set 2 – Moderate

### Q1. List emails, names, and genres of all Rock music listeners.

#### Method 1

```sql
SELECT DISTINCT email, first_name, last_name
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
JOIN invoiceline ON invoice.invoice_id = invoiceline.invoice_id
WHERE track_id IN (
	SELECT track_id FROM track
	JOIN genre ON track.genre_id = genre.genre_id
	WHERE genre.name LIKE 'Rock'
)
ORDER BY email;
```

#### Method 2

```sql
SELECT DISTINCT email AS Email, first_name AS FirstName, last_name AS LastName, genre.name AS Name
FROM customer
JOIN invoice ON invoice.customer_id = customer.customer_id
JOIN invoiceline ON invoiceline.invoice_id = invoice.invoice_id
JOIN track ON track.track_id = invoiceline.track_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
ORDER BY email;
```

### Q2. Top 10 rock bands by track count.

```sql
SELECT artist.artist_id, artist.name, COUNT(artist.artist_id) AS number_of_songs
FROM track
JOIN album ON album.album_id = track.album_id
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs DESC
LIMIT 10;
```

### Q3. Tracks longer than the average song length.

```sql
SELECT name, miliseconds
FROM track
WHERE miliseconds > (
	SELECT AVG(miliseconds) AS avg_track_length FROM track
)
ORDER BY miliseconds DESC;
```

---

## 🧮 Question Set 3 – Advanced

### Q1. Amount spent by each customer on the best-selling artist

```sql
WITH best_selling_artist AS (
	SELECT artist.artist_id, artist.name AS artist_name, 
	       SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
	FROM invoice_line
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN album ON album.album_id = track.album_id
	JOIN artist ON artist.artist_id = album.artist_id
	GROUP BY 1, 2
	ORDER BY total_sales DESC
	LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name,
       SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY 1, 2, 3, 4
ORDER BY amount_spent DESC;
```

### Q2. Most popular genre per country

#### Method 1: Using CTE

```sql
WITH popular_genre AS (
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id,
	       ROW_NUMBER() OVER(PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo
    FROM invoice_line
	JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
	JOIN customer ON customer.customer_id = invoice.customer_id
	JOIN track ON track.track_id = invoice_line.track_id
	JOIN genre ON genre.genre_id = track.genre_id
	GROUP BY 2, 3, 4
)
SELECT * FROM popular_genre WHERE RowNo <= 1;
```

#### Method 2: Using Recursive CTE

```sql
WITH RECURSIVE
	sales_per_country AS (
		SELECT COUNT(*) AS purchases_per_genre, customer.country, genre.name, genre.genre_id
		FROM invoice_line
		JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
		JOIN customer ON customer.customer_id = invoice.customer_id
		JOIN track ON track.track_id = invoice_line.track_id
		JOIN genre ON genre.genre_id = track.genre_id
		GROUP BY 2, 3, 4
	),
	max_genre_per_country AS (
		SELECT country, MAX(purchases_per_genre) AS max_genre_number
		FROM sales_per_country
		GROUP BY country
	)
SELECT s.*
FROM sales_per_country s
JOIN max_genre_per_country m ON s.country = m.country
WHERE s.purchases_per_genre = m.max_genre_number;
```

### Q3. Top-spending customer per country

#### Method 1: Using CTE

```sql
WITH customer_with_country AS (
	SELECT customer.customer_id, first_name, last_name, billing_country, 
	       SUM(total) AS total_spending,
	       ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo
	FROM invoice
	JOIN customer ON customer.customer_id = invoice.customer_id
	GROUP BY 1, 2, 3, 4
)
SELECT * FROM customer_with_country WHERE RowNo <= 1;
```

#### Method 2: Using Recursive CTE

```sql
WITH RECURSIVE 
	customer_with_country AS (
		SELECT customer.customer_id, first_name, last_name, billing_country, SUM(total) AS total_spending
		FROM invoice
		JOIN customer ON customer.customer_id = invoice.customer_id
		GROUP BY 1, 2, 3, 4
	),
	country_max_spending AS (
		SELECT billing_country, MAX(total_spending) AS max_spending
		FROM customer_with_country
		GROUP BY billing_country
	)
SELECT cc.billing_country, cc.total_spending, cc.first_name, cc.last_name, cc.customer_id
FROM customer_with_country cc
JOIN country_max_spending ms
ON cc.billing_country = ms.billing_country
WHERE cc.total_spending = ms.max_spending
ORDER BY 1;
```

---

## 🧰 Technologies Used

* **Database:** PostgreSQL / MySql
* **Concepts Covered:**

  * Joins (INNER, LEFT)
  * Aggregations (`SUM`, `COUNT`, `AVG`)
  * Subqueries
  * Common Table Expressions (CTEs)
  * Recursive Queries
  * Window Functions (`ROW_NUMBER`, `RANK`)
  * GROUP BY and ORDER BY logic

---

## 🧾 Author

**Md. Sarwar Hossen Sagar**
📧 [sarwarhsagar@gmail.com](mailto:sarwarhsagar@gmail.com)
🌐 [Portfolio](https://www.datascienceportfol.io/sarwarhsagar)

---

Would you like me to include **ERD image (as markdown link)** and a **“How to Run the Project”** section (e.g., setup instructions for PostgreSQL)?
That would make your GitHub README more complete and professional.
