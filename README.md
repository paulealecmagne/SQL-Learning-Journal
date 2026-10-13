# SQL-Learning-Journal
This is just my SQL Learning Journal for my class Python for Data and AI

-- Part A — Easy (Items 1–5)

-- Track Names and Composers

SELECT Name, Composer
FROM tracks
LIMIT 5;

-- Customers Located in Germany

SELECT FirstName, LastName, City
FROM customers
WHERE Country = 'Germany';

-- Album Titles in Descending Alphabetical Order (Top 5 Rows)

SELECT Title
FROM albums
ORDER BY Title DESC
LIMIT 5;

-- Customer Count per Support Representative

SELECT SupportRepId, COUNT(*) AS NumCustomers
FROM customers
GROUP BY SupportRepId
ORDER BY NumCustomers DESC;

-- Average Track Price by Genre (Top 5 Highest Prices)

SELECT GenreId, AVG(UnitPrice) AS AvgPrice
FROM tracks
GROUP BY GenreId
ORDER BY AvgPrice DESC
LIMIT 5;

-- Part B — Intermediate (Items 6–10)

-- Track Count per Genre Name (Top 5 Genres)

SELECT g.Name, COUNT(t.TrackId) AS NumTracks
FROM genres g
JOIN tracks t ON g.GenreId = t.GenreId
GROUP BY g.GenreId, g.Name
ORDER BY NumTracks DESC
LIMIT 5;

-- Track, Album, and Artist Details (First 10 Rows)

SELECT 
    t.Name AS TrackName, 
    al.Title AS AlbumTitle, 
    ar.Name AS ArtistName
FROM tracks t
JOIN albums al ON t.AlbumId = al.AlbumId
JOIN artists ar ON al.ArtistId = ar.ArtistId
LIMIT 10;

-- Purchase History for Customer 5

SELECT 
    c.FirstName, 
    c.LastName, 
    t.Name AS TrackName, 
    ii.Quantity, 
    ii.UnitPrice
FROM customers c
JOIN invoices i ON c.CustomerId = i.CustomerId
JOIN invoice_items ii ON i.InvoiceId = ii.InvoiceId
JOIN tracks t ON ii.TrackId = t.TrackId
WHERE c.CustomerId = 5
ORDER BY ii.InvoiceLineId ASC;

-- Playlist Track Counts

SELECT 
    p.Name, 
    COUNT(pt.TrackId) AS NumTracks
FROM playlists p
LEFT JOIN playlist_track pt ON p.PlaylistId = pt.PlaylistId
GROUP BY p.PlaylistId, p.Name
ORDER BY NumTracks DESC;

-- Top 5 Revenue-Generating Countries

SELECT 
    c.Country, 
    SUM(ii.UnitPrice * ii.Quantity) AS Revenue
FROM customers c
JOIN invoices i ON c.CustomerId = i.CustomerId
JOIN invoice_items ii ON i.InvoiceId = ii.InvoiceId
GROUP BY c.Country
ORDER BY Revenue DESC
LIMIT 5;

--Part C — Advanced (Items 11–15)

-- Top 5 Spenders via Scalar Subquery in SELECT

SELECT 
    c.FirstName, 
    c.LastName, 
    (
        SELECT SUM(i.Total) 
        FROM invoices i 
        WHERE i.CustomerId = c.CustomerId
    ) AS TotalSpent
FROM customers c
ORDER BY TotalSpent DESC
LIMIT 5;
12. Tracks Priced Above t

-- Tracks Priced Above the Global Average

SELECT Name, UnitPrice
FROM tracks
WHERE UnitPrice > (
    SELECT AVG(UnitPrice) 
    FROM tracks
);

-- Top 3 Highest-Grossing Genres via CTE

WITH GenreRevenue AS (
    SELECT 
        g.Name AS GenreName, 
        SUM(ii.UnitPrice * ii.Quantity) AS Revenue
    FROM genres g
    JOIN tracks t ON g.GenreId = t.GenreId
    JOIN invoice_items ii ON t.TrackId = ii.TrackId
    GROUP BY g.GenreId, g.Name
)
SELECT GenreName, Revenue
FROM GenreRevenue
ORDER BY Revenue DESC
LIMIT 3;

-- Artists Without Albums via NOT IN Subquery

SELECT ArtistId, Name
FROM artists
WHERE ArtistId NOT IN (
    SELECT DISTINCT ArtistId 
    FROM albums 
    WHERE ArtistId IS NOT NULL
);

-- Above-Average Revenue Customers via CTE

WITH CustomerRevenue AS (
    SELECT 
        c.CustomerId,
        c.FirstName, 
        c.LastName, 
        SUM(ii.UnitPrice * ii.Quantity) AS Revenue
    FROM customers c
    JOIN invoices i ON c.CustomerId = i.CustomerId
    JOIN invoice_items ii ON i.InvoiceId = ii.InvoiceId
    GROUP BY c.CustomerId, c.FirstName, c.LastName
)
SELECT FirstName, LastName, Revenue
FROM CustomerRevenue
WHERE Revenue > (
    SELECT AVG(Revenue) 
    FROM CustomerRevenue
)
ORDER BY Revenue DESC;
