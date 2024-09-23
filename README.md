# UrbanEats Delivery Route Optimization Project

## Business Overview/Problem

UrbanEats faces multiple challenges that need to be addressed:
- Prolonged delivery times, leading to customer dissatisfaction.
- Escalating operational costs due to inefficient routing.
- Inaccurate delivery time estimates frustrating customers.
- High attrition rate among delivery drivers caused by stressful working conditions.

### Project Rationale

Route Optimization is vital for reducing delivery times, minimizing fuel consumption, and lowering operational costs, while enhancing service quality. UrbanEats aims to streamline its food delivery system by implementing route optimization, focusing on high-density urban areas.

**Objectives**:
- Reduce delivery lead times by 20%.
- Cut operational costs by 15%.
- Improve driver satisfaction through stress-free routing strategies.

**Goals**:
- Implement a data-driven route optimization solution using SQL and Power BI.
- Provide real-time order status updates to customers.
- Enhance the overall food delivery experience.

## Data Description

The project utilizes four key datasets:

### 1. **Customer Orders**
- **OrderID** (Primary Key, Integer): Unique order identifier.
- **CustomerID** (Foreign Key, Integer): Identifier for the customer.
- **DeliveryAddress**: Address for food delivery.
- **Latitude** (Decimal): Latitude coordinate of the delivery address.
- **Longitude** (Decimal): Longitude coordinate of the delivery address.
- **OrderTimestamp** (DateTime): Date and time of the order.
- **OrderStatus**: Current status of the order.
- **DriverID** (Foreign Key, Integer): Identifier for the assigned driver.
- **RestaurantID** (Foreign Key, Integer): Identifier for the originating restaurant.

### 2. **Traffic Data**
- **LocationID** (Primary Key, Integer): Unique identifier for each location.
- **LocationName**: Name of the urban location.
- **TrafficDensity** (Decimal): Traffic congestion data for the location.

### 3. **Drivers**
- **DriverID** (Primary Key, Integer): Unique identifier for each driver.
- **DriverName**: Name of the driver.
- **ShiftID** (Integer): Identifier for the driver's shift.
- **ShiftStart** (DateTime): Start time of the shift.
- **ShiftEnd** (DateTime): End time of the shift.

### 4. **Restaurants**
- **RestaurantID** (Primary Key, Integer): Unique identifier for each restaurant.
- **RestaurantName**: Name of the restaurant.
- **Address**: Address of the restaurant.

## Tech Stack

The tools used in this project:
- **MySQL** for data collection, storage, and analysis.
- **Power BI** for data visualization and insight generation.

## Project Scope

1. **Exploratory Data Analysis (EDA)**: Understanding data characteristics and discovering patterns.
2. **Data Transformation**: Preparing the data for analysis.
3. **Data Analysis**: Generating insights using SQL queries.
4. **Data Visualization**: Creating visual representations in Power BI.
5. **Insight Generation**: Interpreting results to enhance decision-making.

## SQL Code Implementation

# Delivery Data Insights Project

This SQL code provides the data processing steps for analyzing delivery performance across multiple restaurants, drivers, and orders. The following SQL script covers creating necessary tables, inserting sample data, and running various queries to extract insights for delivery performance, traffic analysis, and driver attrition.

## SQL Code

#### 1. Schema Creation and Table Setup
```sql
-- Create UrbanEats schema and use it
CREATE SCHEMA UrbanEats;
USE UrbanEats;

-- Create CustomerOrders table
CREATE TABLE CustomerOrders (
    OrderID INT,
    CustomerID INT,
    DeliveryAddress VARCHAR(255),
    Latitude DECIMAL(10, 7),
    Longitude DECIMAL(10, 7),
    OrderTimestamp DATETIME,
    OrderStatus VARCHAR(50),
    DriverID INT,
    RestaurantID INT,
    LocationID INT,
    DistanceKm DECIMAL(6, 3),
    DeliveryHours DECIMAL(5, 2),
    TimeTakenToDeliver TIME,
    DeliveryTime DATETIME
);

-- Load customer orders from a CSV file
SET GLOBAL local_infile = 1;

LOAD DATA INFILE 'C:/ProgramData/MySQL/MySQL Server 8.0/Uploads/Urbaneats/customer_orders_realistic.csv'
INTO TABLE CustomerOrders
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 ROWS
(OrderID, CustomerID, DeliveryAddress, Latitude, Longitude, @OrderTimestamp, OrderStatus, DriverID, RestaurantID, LocationID, DistanceKm, DeliveryHours, TimeTakenToDeliver, @DeliveryTime)
SET 
    OrderTimestamp = STR_TO_DATE(@OrderTimestamp, '%d/%m/%Y %H:%i'),
    DeliveryTime = STR_TO_DATE(@DeliveryTime, '%d/%m/%Y %H:%i');

-- Check configuration
SHOW VARIABLES LIKE 'local_infile';

```
#### 2. Data Exploration
```sql
-- Display all tables in the database
SHOW TABLES;

-- Preview data from CustomerOrders table
SELECT * FROM CustomerOrders;

-- Count the total number of records
SELECT COUNT(*) AS Orders_Count FROM CustomerOrders;

-- Describe the structure of the drivers_realistic table
DESCRIBE drivers_realistic;
```
#### 3. Data Cleaning and Transformation
```sql
-- Update date formats for drivers_realistic table
UPDATE drivers_realistic
SET shiftstart = STR_TO_DATE(shiftstart, '%d/%m/%Y %H:%i'),
    shiftend = STR_TO_DATE(shiftend, '%d/%m/%Y %H:%i');

-- Modify columns to proper datetime format
ALTER TABLE drivers_realistic
MODIFY shiftstart DATETIME,
MODIFY shiftend DATETIME;

-- Update datetime format in CustomerOrders
UPDATE CustomerOrders
SET OrderTimestamp = STR_TO_DATE(OrderTimestamp, '%d/%m/%Y %H:%i'),
    DeliveryTime = STR_TO_DATE(DeliveryTime, '%d/%m/%Y %H:%i');

-- Alter columns to datetime format
ALTER TABLE CustomerOrders
MODIFY COLUMN OrderTimestamp DATETIME, 
MODIFY COLUMN DeliveryTime DATETIME;
```
#### 4. Feature Engineering
```sql
-- Estimated Travel Time based on distance and traffic
SELECT c.orderid, 
       c.distancekm, 
       ROUND(c.distancekm * (1 + t.trafficdensity) / 100) AS Estimated_Time_Taken_Mins
FROM customerorders c
JOIN traffic_data_realistic t 
  ON c.locationid = t.locationid;

-- Calculate driver shift length
SELECT driverid, shiftstart, shiftend, 
       TIMESTAMPDIFF(HOUR, shiftstart, shiftend) AS Shift_Length_Hr
FROM drivers_realistic;

-- Average Delivery Time per Restaurant
SELECT r.restaurantid, r.restaurantname, 
       AVG(c.deliveryhours) AS AVG_Delivery_Time
FROM restaurants_realistic AS r
JOIN customerorders AS c ON r.restaurantid = c.restaurantid
GROUP BY r.restaurantid, r.restaurantname;

-- Identify busy driver periods
SELECT driverid, 
       EXTRACT(HOUR FROM ordertimestamp) AS OrderHours, 
       COUNT(*) AS Numberoforders 
FROM customerorders
GROUP BY driverid, OrderHours;
```
#### 5. Preliminary Analysis
```sql
-- Average, Min, Max Delivery Time
SELECT AVG(DeliveryHours) AS AvgDeliveryTime, 
       MIN(DeliveryHours) AS MinDeliveryTime, 
       MAX(DeliveryHours) AS MaxDeliveryTime
FROM customerorders;

-- Frequency of Delivery Status
SELECT orderstatus, COUNT(*) AS Status_Count 
FROM customerorders 
GROUP BY orderstatus;
```
#### 4. Identifying Peak Hours and Days
```sql
-- Peak hours for orders
SELECT EXTRACT(HOUR FROM ordertimestamp) AS Hours_of_Day, 
       COUNT(*) AS Order_Count
FROM customerorders 
GROUP BY Hours_of_Day;

-- Peak days for orders
SELECT DAYNAME(ordertimestamp) AS Day_of_Week, 
       COUNT(*) AS Order_Count
FROM customerorders 
GROUP BY Day_of_Week;
```
#### 5. Traffic and Delivery Time Correlation
```sql
-- Correlation between traffic density and average delivery time
SELECT t.trafficdensity, 
       AVG(c.deliveryhours) AS Avg_Delivery_Hour
FROM traffic_data_realistic t
JOIN customerorders c ON t.locationid = c.locationid
GROUP BY t.trafficdensity;
```
# Delivery Service Analysis (Power BI)

The analysis is based on a delivery platform handling orders from multiple restaurants with a fleet of 50 drivers.

### Key Metrics
- **Total Orders**: 1,000
- **Total Restaurants**: 20
- **Orders Delivered**: 347
- **Orders Pending**: 653
- **Average Delivery Hours**: 0.15 hours (9 minutes)
- **Total Drivers**: 50

### Order Insights
- **Order Status**:
  - 65.3% Delivered
  - 34.7% Pending
- **Top Drivers** (by Orders Delivered):
  - Thomas McDonald: 36
  - Dawn Ferguson: 29
  - Melody Jenkins: 28
- **Order Trends**: 
  - Highest orders on **Tuesday** (181), decreasing to 115 by **Thursday**.
- **Top Restaurants** (by Orders):
  - Nguyen-Lopez: 68
  - Ryan, Alexander and Willis: 67

## Traffic Density Analysis
- **Average Traffic Density**: 69.85 vehicles per minute
- **Busiest Hour**: 7 AM
- **Top Restaurants with Highest Delays**:
  - Ryan, Alexander and Willis: 10.8 minutes
  - Nguyen-Lopez: 10.6 minutes
  - Mcdaniel-Harrington: 9.9 minutes

### Performance Insights
- **Delivery Performance**: Peaks at **8.0** (2 PM) and drops to **3.3** (9 PM).
- **Route with Highest Traffic**: Congestion on a mapped route is likely affecting deliveries.

## Driver Attrition Analysis
- **Average Shift Duration**: 7.06 hours
- **Max Shift Duration**: 8 hours
- **Min Shift Duration**: 6 hours
- **Drivers with Most Shifts**: Dawn Ferguson, Michelle Ballard, Thomas McDonald (8 hours each)
- **Drivers with Least Shifts**: Charles Carr, Lindsay Pierce (7 hours each)
- **Drivers with Most Delays**: 
  - Maria Stephenson: 0.132 hours (~8 minutes)
  - Andrea Jones: 0.119 hours
- **Drivers with Least Delays**: 
  - Cynthia Vasquez: 0.201 hours
  - Eric Garcia: 0.191 hours

---

## Summary of Insights
- **High Pending Orders**: 653 orders remain pending, signaling a need for operational adjustments.
- **Driver Efficiency**: Top drivers are delivering more efficiently, but there is variation in shift length and delays.
- **Impact of Traffic**: High traffic density and key traffic routes are affecting delivery times.
- **Order Peaks**: Orders peak on Tuesdays and decrease by the end of the week, suggesting resource reallocation opportunities.



