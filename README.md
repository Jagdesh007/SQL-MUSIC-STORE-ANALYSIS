🎵 SQL Music Store Data Analysis

A complete end-to-end SQL project exploring customer behavior, sales trends, genre popularity, and artist performance using the Music Store database. This project answers beginner, intermediate, and advanced analytics questions using optimized SQL queries.

🚀 Project Overview

This SQL project analyzes a digital music store’s dataset consisting of customers, invoices, tracks, artists, genres, and employees.
You will learn to write JOINs, aggregations, subqueries, CTEs, window functions, and real-world business logic queries.

📂 Dataset Entities

Employee
Customer
Invoice
InvoiceLine
Track
Genre
Artist
Album

🟢 QUESTION SET 1 — EASY
1️⃣ Senior-most employee based on job title
SELECT *
FROM Employee
ORDER BY Title DESC
LIMIT 1;

2️⃣ Countries with the most invoices
SELECT BillingCountry, COUNT(*) AS InvoiceCount
FROM Invoice
GROUP BY BillingCountry
ORDER BY InvoiceCount DESC;

3️⃣ Top 3 invoice totals
SELECT Total
FROM Invoice
ORDER BY Total DESC
LIMIT 3;

4️⃣ City with highest revenue (sum of invoices)
SELECT BillingCity, SUM(Total) AS TotalRevenue
FROM Invoice
GROUP BY BillingCity
ORDER BY TotalRevenue DESC
LIMIT 1;

5️⃣ Best customer (highest spending)
SELECT c.CustomerId, c.FirstName, c.LastName, SUM(i.Total) AS TotalSpent
FROM Customer c
JOIN Invoice i ON c.CustomerId = i.CustomerId
GROUP BY c.CustomerId
ORDER BY TotalSpent DESC
LIMIT 1;

🟡 QUESTION SET 2 — MODERATE
1️⃣ Rock music listeners (email, first, last name)
SELECT DISTINCT c.Email, c.FirstName, c.LastName, g.Name AS Genre
FROM Customer c
JOIN Invoice i ON c.CustomerId = i.CustomerId
JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
JOIN Track t ON il.TrackId = t.TrackId
JOIN Genre g ON t.GenreId = g.GenreId
WHERE g.Name = 'Rock'
ORDER BY c.Email ASC;

2️⃣ Top 10 artists with the most rock tracks
SELECT ar.Name AS ArtistName, COUNT(t.TrackId) AS RockTrackCount
FROM Artist ar
JOIN Album al ON ar.ArtistId = al.ArtistId
JOIN Track t ON al.AlbumId = t.AlbumId
JOIN Genre g ON t.GenreId = g.GenreId
WHERE g.Name = 'Rock'
GROUP BY ar.Name
ORDER BY RockTrackCount DESC
LIMIT 10;

3️⃣ Tracks longer than average song length
SELECT Name, Milliseconds
FROM Track
WHERE Milliseconds > (SELECT AVG(Milliseconds) FROM Track)
ORDER BY Milliseconds DESC;

🔴 QUESTION SET 3 — ADVANCED
1️⃣ Spend of each customer on each artist
WITH ArtistInvoices AS (
    SELECT c.CustomerId,
           c.FirstName,
           c.LastName,
           ar.Name AS ArtistName,
           SUM(il.UnitPrice * il.Quantity) AS TotalSpent
    FROM Invoice i
    JOIN Customer c ON i.CustomerId = c.CustomerId
    JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
    JOIN Track t ON il.TrackId = t.TrackId
    JOIN Album al ON t.AlbumId = al.AlbumId
    JOIN Artist ar ON al.ArtistId = ar.ArtistId
    GROUP BY c.CustomerId, ArtistName
)
SELECT *
FROM ArtistInvoices
ORDER BY TotalSpent DESC;

2️⃣ Most popular genre in each country
WITH GenreCount AS (
    SELECT i.BillingCountry,
           g.Name AS Genre,
           COUNT(il.InvoiceLineId) AS Purchases
    FROM Invoice i
    JOIN InvoiceLine il ON i.InvoiceId = il.InvoiceId
    JOIN Track t ON il.TrackId = t.TrackId
    JOIN Genre g ON t.GenreId = g.GenreId
    GROUP BY i.BillingCountry, g.Name
),
MaxGenre AS (
    SELECT BillingCountry, MAX(Purchases) AS MaxPurchases
    FROM GenreCount
    GROUP BY BillingCountry
)
SELECT gc.BillingCountry, gc.Genre, gc.Purchases
FROM GenreCount gc
JOIN MaxGenre mg ON gc.BillingCountry = mg.BillingCountry
                AND gc.Purchases = mg.MaxPurchases
ORDER BY gc.BillingCountry;

3️⃣ Top-spending customer in each country
WITH CustomerSpending AS (
    SELECT c.CustomerId,
           c.FirstName,
           c.LastName,
           c.Country,
           SUM(i.Total) AS TotalSpent
    FROM Customer c
    JOIN Invoice i ON c.CustomerId = i.CustomerId
    GROUP BY c.CustomerId
),
MaxSpending AS (
    SELECT Country, MAX(TotalSpent) AS MaxSpent
    FROM CustomerSpending
    GROUP BY Country
)
SELECT cs.Country, cs.FirstName, cs.LastName, cs.TotalSpent
FROM CustomerSpending cs
JOIN MaxSpending ms ON cs.Country = ms.Country
                   AND cs.TotalSpent = ms.MaxSpent
ORDER BY cs.Country;
