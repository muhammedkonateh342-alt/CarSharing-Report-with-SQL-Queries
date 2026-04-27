# CarSharing Report with SQL Queries

## Project Overview
This project analyzes a car-sharing dataset from a car-sharing company. The dataset contains information about customer demand rates between **January 2017 and August 2018**, collected on an hourly basis. The data includes time information (date, hour, season) and weather data (weather condition, temperature, humidity, wind speed).

The **demand** column represents the customer's willingness to rent a car at a specific time. Higher demand rates indicate greater willingness to rent.

---

## Tools Used
- **Google Sheets** — Data cleaning, manipulation and table creation
- **Microsoft SQL Server** — Database creation, relationships and SQL queries
- **GitHub** — Documentation and collaboration

---

## Database Structure

The database called **carsharing** contains 4 tables:

| Table | Description |
|---|---|
| CarSharing_df | Main table with demand, humidity, windspeed and foreign keys |
| Temperature | Temperature details including temp, temp_feel and temp_category |
| WEATHER | Weather condition descriptions and codes |
| Time | Timestamp, season, hour, weekday and month information |

### Entity Relationship Diagram (ERD)
The tables are linked as follows:
- **CarSharing_df** → **Temperature** via `Temp_Code`
- **CarSharing_df** → **WEATHER** via `Weather_Code`
- **CarSharing_df** → **Time** via `id`

---

## Part 1: Google Sheets Data Manipulation

The following steps were completed in Google Sheets:

1. **Filled missing values** in `temp` and `temp_feel` columns using AVERAGE function
2. **Added `temp_category` column** — Cold (temp_feel < 10), Mild (10-25), Hot (> 25)
3. **Added `temp_code` column** — Concatenated temp, temp_feel and temp_category (e.g. 9.84-14.395-Mild)
4. **Added `weather_code` column** — Clear or partly cloudy=1, Mist=2, Light snow or rain=3, Heavy rain=4
5. **Added `hour`, `weekday name`, and `month name` columns** extracted from timestamp
6. **Created `weather` sheet** — Contains weather and weather_code columns (duplicates removed)
7. **Created `temperature` sheet** — Contains temp, temp_feel, temp_category, temp_code (duplicates removed)
8. **Created `time` sheet** — Contains id, timestamp, season, hour, weekday name, month name

---

## Part 2: Database Management

### Creating the Database
```sql
CREATE DATABASE carsharing;
```

### Creating Relationships
```sql
-- Primary Keys
ALTER TABLE [Temperature]
ADD CONSTRAINT PK_Temperature PRIMARY KEY (Temp_Code);

ALTER TABLE [WEATHER]
ADD CONSTRAINT PK_Weather PRIMARY KEY (Weather_code);

ALTER TABLE [Time]
ADD CONSTRAINT PK_Time PRIMARY KEY (id);

ALTER TABLE [CarSharing_df]
ADD CONSTRAINT PK_CarSharing PRIMARY KEY (id);

-- Foreign Keys
ALTER TABLE [CarSharing_df]
ADD CONSTRAINT FK_Temperature
FOREIGN KEY (Temp_Code) REFERENCES [Temperature](Temp_Code);

ALTER TABLE [CarSharing_df]
ADD CONSTRAINT FK_Weather
FOREIGN KEY (Weather_Code) REFERENCES [WEATHER](Weather_code);

ALTER TABLE [CarSharing_df]
ADD CONSTRAINT FK_Time
FOREIGN KEY (id) REFERENCES [Time](id);
```

---

## Part 3: SQL Queries — Linda's Business Questions

### Question (a): Highest Demand Rate in 2017
*Which date and time did we have the highest demand rate in 2017?*

```sql
-- Question (a): Find the date and time with highest demand rate in 2017
SELECT TOP 1
    [Time].timestamp,
    [CarSharing_df].demand
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
WHERE YEAR([Time].timestamp) = 2017
ORDER BY [CarSharing_df].demand DESC;
```

**Result:**
| timestamp | demand |
|---|---|
| 2017-06-15 17:00:00 | 6.46 |

**Finding:** The highest demand rate in 2017 was on **15th June 2017 at 5:00 PM** with a demand rate of **6.46**.

---

### Question (b): Highest and Lowest Average Demand in 2017
*Weekday, month and season with highest and lowest average demand throughout 2017.*

```sql
-- Question (b): Weekday, Month and Season with HIGHEST average demand in 2017
SELECT TOP 1
    [Time].Weekday_Name,
    [Time].Month_Name,
    [Time].season,
    ROUND(AVG([CarSharing_df].demand), 2) AS Avg_Demand
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
WHERE YEAR([Time].timestamp) = 2017
GROUP BY [Time].Weekday_Name, [Time].Month_Name, [Time].season
ORDER BY Avg_Demand DESC;

-- Question (b): Weekday, Month and Season with LOWEST average demand in 2017
SELECT TOP 1
    [Time].Weekday_Name,
    [Time].Month_Name,
    [Time].season,
    ROUND(AVG([CarSharing_df].demand), 2) AS Avg_Demand
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
WHERE YEAR([Time].timestamp) = 2017
GROUP BY [Time].Weekday_Name, [Time].Month_Name, [Time].season
ORDER BY Avg_Demand ASC;
```

**Result:**
| Demand Type | Weekday | Month | Season | Avg Demand |
|---|---|---|---|---|
| Highest | Sunday | July | Fall | 4.99 |
| Lowest | Monday | January | Spring | 3.05 |

**Finding:** **Sunday in July (Fall season)** had the highest average demand while **Monday in January (Spring season)** had the lowest.

---

### Question (c): Average Demand per Hour for Highest and Lowest Weekdays
*Average demand at different hours for Sunday and Monday throughout 2017.*

```sql
-- Question (c): Average demand per hour for SUNDAY in 2017
SELECT 
    [Time].Hour,
    ROUND(AVG([CarSharing_df].demand), 2) AS Avg_Demand
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
WHERE YEAR([Time].timestamp) = 2017
AND [Time].Weekday_Name = 'Sunday'
GROUP BY [Time].Hour
ORDER BY Avg_Demand DESC;

-- Question (c): Average demand per hour for MONDAY in 2017
SELECT 
    [Time].Hour,
    ROUND(AVG([CarSharing_df].demand), 2) AS Avg_Demand
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
WHERE YEAR([Time].timestamp) = 2017
AND [Time].Weekday_Name = 'Monday'
GROUP BY [Time].Hour
ORDER BY Avg_Demand DESC;
```

**Finding:** 
- **Sunday** peak demand was at **3:00 PM (Hour 15)** with avg demand of 5.54
- **Monday** peak demand was at **1:00 PM (Hour 13)** with avg demand of 5.64

---

### Question (d): Weather Analysis for 2017

#### Temperature Category Distribution
```sql
-- Question (d): Count of Cold, Mild and Hot weather in 2017
SELECT 
    [Temperature].Temp_Category,
    COUNT(*) AS Total_Hours,
    ROUND(AVG([CarSharing_df].demand), 2) AS Avg_Demand
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
JOIN [Temperature] ON [CarSharing_df].Temp_Code = [Temperature].Temp_Code
WHERE YEAR([Time].timestamp) = 2017
GROUP BY [Temperature].Temp_Category
ORDER BY Total_Hours DESC;
```

**Result:**
| Temp Category | Total Hours | Avg Demand |
|---|---|---|
| Mild | 2734 | 4.02 |
| Hot | 2296 | 4.80 |
| Cold | 390 | 3.19 |

#### Most Prevalent Weather Condition
```sql
-- Question (d): Most prevalent weather condition in 2017
SELECT 
    [WEATHER].weather,
    COUNT(*) AS Total_Hours
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
JOIN [WEATHER] ON [CarSharing_df].Weather_Code = [WEATHER].Weather_code
WHERE YEAR([Time].timestamp) = 2017
GROUP BY [WEATHER].weather
ORDER BY Total_Hours DESC;
```

**Result:**
| Weather | Total Hours |
|---|---|
| Clear or partly cloudy | 3582 |
| Mist | 1366 |
| Light snow or rain | 472 |

#### Wind Speed Statistics per Month
```sql
-- Question (d): Wind speed statistics per month in 2017
SELECT 
    [Time].Month_Name,
    ROUND(AVG([CarSharing_df].windspeed), 2) AS Avg_Windspeed,
    ROUND(MAX([CarSharing_df].windspeed), 2) AS Max_Windspeed,
    ROUND(MIN([CarSharing_df].windspeed), 2) AS Min_Windspeed
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
WHERE YEAR([Time].timestamp) = 2017
GROUP BY [Time].Month_Name
ORDER BY MIN([Time].timestamp) ASC;
```

#### Humidity Statistics per Month
```sql
-- Question (d): Humidity statistics per month in 2017
SELECT 
    [Time].Month_Name,
    ROUND(AVG([CarSharing_df].humidity), 2) AS Avg_Humidity,
    ROUND(MAX([CarSharing_df].humidity), 2) AS Max_Humidity,
    ROUND(MIN([CarSharing_df].humidity), 2) AS Min_Humidity
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
WHERE YEAR([Time].timestamp) = 2017
GROUP BY [Time].Month_Name
ORDER BY MIN([Time].timestamp) ASC;
```

#### Average Demand by Temperature Category
```sql
-- Question (d): Average demand for each temperature category in 2017
SELECT 
    [Temperature].Temp_Category,
    ROUND(AVG([CarSharing_df].demand), 2) AS Avg_Demand
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
JOIN [Temperature] ON [CarSharing_df].Temp_Code = [Temperature].Temp_Code
WHERE YEAR([Time].timestamp) = 2017
GROUP BY [Temperature].Temp_Category
ORDER BY Avg_Demand DESC;
```

**Result:**
| Temp Category | Avg Demand |
|---|---|
| Hot | 4.80 |
| Mild | 4.02 |
| Cold | 3.19 |

**Finding:** 2017 was mostly **Mild** weather with **Clear or partly cloudy** conditions being most common. Demand was highest during **Hot** weather conditions.

---

### Question (e): Weather Analysis for July 2017 (Highest Demand Month)

*July 2017 had the highest average demand of 4.79*

```sql
-- Question (e): Temperature category for July 2017
SELECT 
    [Temperature].Temp_Category,
    COUNT(*) AS Total_Hours,
    ROUND(AVG([CarSharing_df].demand), 2) AS Avg_Demand
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
JOIN [Temperature] ON [CarSharing_df].Temp_Code = [Temperature].Temp_Code
WHERE YEAR([Time].timestamp) = 2017
AND [Time].Month_Name = 'July'
GROUP BY [Temperature].Temp_Category
ORDER BY Total_Hours DESC;

-- Question (e): Most prevalent weather condition in July 2017
SELECT 
    [WEATHER].weather,
    COUNT(*) AS Total_Hours
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
JOIN [WEATHER] ON [CarSharing_df].Weather_Code = [WEATHER].Weather_code
WHERE YEAR([Time].timestamp) = 2017
AND [Time].Month_Name = 'July'
GROUP BY [WEATHER].weather
ORDER BY Total_Hours DESC;

-- Question (e): Wind speed statistics for July 2017
SELECT 
    [Time].Month_Name,
    ROUND(AVG([CarSharing_df].windspeed), 2) AS Avg_Windspeed,
    ROUND(MAX([CarSharing_df].windspeed), 2) AS Max_Windspeed,
    ROUND(MIN([CarSharing_df].windspeed), 2) AS Min_Windspeed
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
WHERE YEAR([Time].timestamp) = 2017
AND [Time].Month_Name = 'July'
GROUP BY [Time].Month_Name;

-- Question (e): Humidity statistics for July 2017
SELECT 
    [Time].Month_Name,
    ROUND(AVG([CarSharing_df].humidity), 2) AS Avg_Humidity,
    ROUND(MAX([CarSharing_df].humidity), 2) AS Max_Humidity,
    ROUND(MIN([CarSharing_df].humidity), 2) AS Min_Humidity
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
WHERE YEAR([Time].timestamp) = 2017
AND [Time].Month_Name = 'July'
GROUP BY [Time].Month_Name;

-- Question (e): Average demand by temperature category for July 2017
SELECT 
    [Temperature].Temp_Category,
    ROUND(AVG([CarSharing_df].demand), 2) AS Avg_Demand
FROM [CarSharing_df]
JOIN [Time] ON [CarSharing_df].id = [Time].id
JOIN [Temperature] ON [CarSharing_df].Temp_Code = [Temperature].Temp_Code
WHERE YEAR([Time].timestamp) = 2017
AND [Time].Month_Name = 'July'
GROUP BY [Temperature].Temp_Category
ORDER BY Avg_Demand DESC;
```

**July 2017 Results:**
| Metric | Value |
|---|---|
| Most common temp | Hot (448 hours) |
| Most common weather | Clear or partly cloudy (385 hours) |
| Avg Wind Speed | 12.03 |
| Max Wind Speed | 57 |
| Avg Humidity | 60.3 |
| Max Humidity | 94 |
| Demand (Mild) | 4.81 |
| Demand (Hot) | 4.79 |

---

## Key Findings

1. **Peak demand** occurred on **15th June 2017 at 5:00 PM** with a rate of 6.46
2. **Sunday in July** had the highest average demand (4.99) while **Monday in January** had the lowest (3.05)
3. **Hot weather** drives the highest demand (4.80) compared to Mild (4.02) and Cold (3.19)
4. **2017 was mostly Mild** weather with Clear or partly cloudy conditions dominating
5. **July** was the highest demand month with almost entirely Hot weather conditions
6. Peak rental hours are between **1PM and 5PM** regardless of weekday

---

## Data Tables
All tables are stored in Google Drive:
[Google Drive Folder - CarSharing Tables](#) ← Add your Google Drive link here

---

*Report prepared as part of SIWES Project 1 — CarSharing Analysis*
