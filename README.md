# Базы данных: SQL-задачи

## Описание проекта
Данный репозиторий содержит решения 13 задач по четырём тематическим базам данных:
- Транспортные средства (автомобили, мотоциклы, велосипеды)
- Автомобильные гонки
- Бронирование отелей
- Структура организации (сотрудники, проекты, задачи)

## Цели и задачи
- Закрепить навыки написания SQL-запросов различной сложности
- Использовать JOIN, GROUP BY, подзапросы, CTE (WITH), рекурсивные запросы
- Работать с агрегацией, конкатенацией строк, условной логикой

## Структура базы данных
Каждая папка содержит:
- `schema.sql` – создание таблиц (совместимо с MySQL и PostgreSQL)
- `data.sql` – тестовые данные
- `Readme.md` – снизу представлено решение для каждой из задач

### Схемы связей
- **vehicles**: Vehicle (maker, model, type) – родитель, Car/Motorcycle/Bicycle – наследники по model.
- **gonki**: Classes → Cars, Cars → Results, Races → Results.
- **hotels**: Hotel → Room → Booking ← Customer.
- **organization**: Departments, Roles, Employees (самореференция), Projects (по отделам), Tasks (назначены сотрудникам).

## Инструкция по запуску и тестированию

### Для MySQL (через XAMPP)
1. Установите XAMPP, запустите Apache и MySQL.
2. Откройте phpMyAdmin, создайте новую базу данных (например, `vehicles_db`).
3. Перейдите в раздел SQL, выполните скрипт `schema.sql` для создания таблиц.
4. Выполните скрипт `data.sql` для наполнения данными.
5. Выполните каждый `taskN.sql` для проверки результата.
6. Сравните вывод с ожидаемым (приведён в условии задачи).

### Для PostgreSQL (через PgAdmin)
1. Установите PostgreSQL и PgAdmin.
2. Создайте новую базу данных.
3. Откройте Query Tool, выполните `schema.sql` (адаптированный для PostgreSQL, используя SERIAL).
4. Выполните `data.sql`.
5. Выполните файлы с решениями.

## Результаты выполнения

# Задача 1
```sql
SELECT 
    v.maker, 
    m.model
FROM Vehicle v
JOIN Motorcycle m ON v.model = m.model
WHERE m.horsepower > 150 
  AND m.price < 20000 
  AND m.type = 'Sport'
ORDER BY m.horsepower DESC;
```
# Задача 2
```sql
SELECT 
    v.maker, 
    c.model, 
    c.horsepower, 
    c.engine_capacity, 
    'Car' AS vehicle_type
FROM Vehicle v
JOIN Car c ON v.model = c.model
WHERE c.horsepower > 150 
  AND c.engine_capacity < 3 
  AND c.price < 35000

UNION

SELECT 
    v.maker, 
    mc.model, 
    mc.horsepower, 
    mc.engine_capacity, 
    'Motorcycle' AS vehicle_type
FROM Vehicle v
JOIN Motorcycle mc ON v.model = mc.model
WHERE mc.horsepower > 150 
  AND mc.engine_capacity < 1.5 
  AND mc.price < 20000

UNION

SELECT 
    v.maker, 
    b.model, 
    NULL AS horsepower, 
    NULL AS engine_capacity, 
    'Bicycle' AS vehicle_type
FROM Vehicle v
JOIN Bicycle b ON v.model = b.model
WHERE b.gear_count > 18 
  AND b.price < 4000

ORDER BY horsepower DESC;
```
# Задача 3
```sql
WITH CarStats AS (
    SELECT 
        c.name AS car_name,
        c.class AS car_class,
        AVG(r.position) AS average_position,
        COUNT(r.race) AS race_count
    FROM Cars c
    JOIN Results r ON c.name = r.car
    GROUP BY c.name, c.class
),
ClassMinAvg AS (
    SELECT 
        car_class,
        MIN(average_position) AS min_avg_position
    FROM CarStats
    GROUP BY car_class
)
SELECT 
    cs.car_name,
    cs.car_class,
    ROUND(cs.average_position, 4) AS average_position,
    cs.race_count
FROM CarStats cs
JOIN ClassMinAvg cma ON cs.car_class = cma.car_class 
                     AND cs.average_position = cma.min_avg_position
ORDER BY cs.average_position;
```
# Задача 4
```sql
WITH CarStats AS (
    SELECT 
        c.name AS car_name,
        c.class AS car_class,
        AVG(r.position) AS average_position,
        COUNT(r.race) AS race_count,
        cl.country AS car_country
    FROM Cars c
    JOIN Results r ON c.name = r.car
    JOIN Classes cl ON c.class = cl.class
    GROUP BY c.name, c.class, cl.country
)
SELECT 
    car_name,
    car_class,
    ROUND(average_position, 4) AS average_position,
    race_count,
    car_country
FROM CarStats
ORDER BY average_position, car_name
LIMIT 1;
```
# Задача 5
```sql
WITH CarStats AS (
    SELECT 
        c.name AS car_name,
        c.class AS car_class,
        AVG(r.position) AS average_position,
        COUNT(r.race) AS race_count,
        cl.country AS car_country
    FROM Cars c
    JOIN Results r ON c.name = r.car
    JOIN Classes cl ON c.class = cl.class
    GROUP BY c.name, c.class, cl.country
),
ClassAvg AS (
    SELECT 
        car_class,
        AVG(average_position) AS class_avg_position
    FROM CarStats
    GROUP BY car_class
),
MinClassAvg AS (
    SELECT MIN(class_avg_position) AS min_avg
    FROM ClassAvg
),
TargetClasses AS (
    SELECT car_class
    FROM ClassAvg
    WHERE class_avg_position = (SELECT min_avg FROM MinClassAvg)
),
ClassTotalRaces AS (
    SELECT 
        car_class,
        SUM(race_count) AS total_races
    FROM CarStats
    GROUP BY car_class
)
SELECT 
    cs.car_name,
    cs.car_class,
    ROUND(cs.average_position, 4) AS average_position,
    cs.race_count,
    cs.car_country,
    ctr.total_races
FROM CarStats cs
JOIN TargetClasses tc ON cs.car_class = tc.car_class
JOIN ClassTotalRaces ctr ON cs.car_class = ctr.car_class
ORDER BY cs.average_position;
```
# Задача 6
```sql
WITH CarStats AS (
    SELECT 
        c.name AS car_name,
        c.class AS car_class,
        AVG(r.position) AS average_position,
        COUNT(r.race) AS race_count,
        cl.country AS car_country
    FROM Cars c
    JOIN Results r ON c.name = r.car
    JOIN Classes cl ON c.class = cl.class
    GROUP BY c.name, c.class, cl.country
),
ClassAvg AS (
    SELECT 
        car_class,
        AVG(average_position) AS class_avg_position,
        COUNT(*) AS cars_in_class
    FROM CarStats
    GROUP BY car_class
    HAVING COUNT(*) >= 2
)
SELECT 
    cs.car_name,
    cs.car_class,
    ROUND(cs.average_position, 4) AS average_position,
    cs.race_count,
    cs.car_country
FROM CarStats cs
JOIN ClassAvg ca ON cs.car_class = ca.car_class
WHERE cs.average_position < ca.class_avg_position
ORDER BY cs.car_class, cs.average_position;
```
# Задача 7
```sql
SELECT 
    cs.car_name,
    cs.car_class,
    ROUND(cs.average_position, 4) AS average_position,
    cs.race_count,
    cs.car_country,
    ci.total_races,
    ci.total_cars AS low_position_count
FROM (
    SELECT 
        c.name AS car_name,
        c.class AS car_class,
        AVG(r.position) AS average_position,
        COUNT(r.race) AS race_count,
        cl.country AS car_country
    FROM Cars c
    JOIN Results r ON c.name = r.car
    JOIN Classes cl ON c.class = cl.class
    GROUP BY c.name, c.class, cl.country
) cs
JOIN (
    SELECT 
        c.class AS car_class,
        SUM(r.race_count) AS total_races,
        COUNT(*) AS total_cars
    FROM Cars c
    JOIN (
        SELECT car, COUNT(*) AS race_count
        FROM Results
        GROUP BY car
    ) r ON c.name = r.car
    GROUP BY c.class
) ci ON cs.car_class = ci.car_class
WHERE cs.average_position > 3.0
ORDER BY ci.total_cars DESC, cs.car_class, cs.average_position;
```
# Задача 8
```sql
SELECT 
    c.name,
    c.email,
    c.phone,
    COUNT(b.ID_booking) AS total_bookings,
    GROUP_CONCAT(DISTINCT h.name ORDER BY h.name SEPARATOR ', ') AS hotel_list,
    ROUND(AVG(DATEDIFF(b.check_out_date, b.check_in_date)), 4) AS avg_stay_days
FROM Customer c
JOIN Booking b ON c.ID_customer = b.ID_customer
JOIN Room r ON b.ID_room = r.ID_room
JOIN Hotel h ON r.ID_hotel = h.ID_hotel
GROUP BY c.ID_customer
HAVING COUNT(DISTINCT h.ID_hotel) >= 2 AND COUNT(b.ID_booking) > 2
ORDER BY total_bookings DESC;
```
# Задача 9
```sql
SELECT 
    c.ID_customer,
    c.name,
    COUNT(b.ID_booking) AS total_bookings,
    SUM(r.price) AS total_spent,
    COUNT(DISTINCT h.ID_hotel) AS unique_hotels
FROM Customer c
JOIN Booking b ON c.ID_customer = b.ID_customer
JOIN Room r ON b.ID_room = r.ID_room
JOIN Hotel h ON r.ID_hotel = h.ID_hotel
GROUP BY c.ID_customer
HAVING COUNT(b.ID_booking) > 2 
   AND COUNT(DISTINCT h.ID_hotel) > 1 
   AND SUM(r.price) > 500
ORDER BY total_spent ASC;
```
# Задача 10
```sql
SELECT 
    c.ID_customer,
    c.name,
    CASE 
        WHEN MAX(CASE WHEN cat.category = 'Дорогой' THEN 1 ELSE 0 END) = 1 THEN 'Дорогой'
        WHEN MAX(CASE WHEN cat.category = 'Средний' THEN 1 ELSE 0 END) = 1 THEN 'Средний'
        ELSE 'Дешевый'
    END AS preferred_hotel_type,
    GROUP_CONCAT(DISTINCT cat.name ORDER BY cat.name SEPARATOR ', ') AS visited_hotels
FROM (
    SELECT 
        h.ID_hotel,
        h.name,
        CASE 
            WHEN AVG(r.price) < 175 THEN 'Дешевый'
            WHEN AVG(r.price) <= 300 THEN 'Средний'
            ELSE 'Дорогой'
        END AS category
    FROM Hotel h
    JOIN Room r ON h.ID_hotel = r.ID_hotel
    GROUP BY h.ID_hotel
) cat
JOIN Room r ON cat.ID_hotel = r.ID_hotel
JOIN Booking b ON r.ID_room = b.ID_room
JOIN Customer c ON b.ID_customer = c.ID_customer
GROUP BY c.ID_customer
ORDER BY 
    CASE 
        WHEN MAX(CASE WHEN cat.category = 'Дорогой' THEN 1 ELSE 0 END) = 1 THEN 3
        WHEN MAX(CASE WHEN cat.category = 'Средний' THEN 1 ELSE 0 END) = 1 THEN 2
        ELSE 1
    END;
```
# Задача 11
```sql
WITH RECURSIVE EmployeeHierarchy AS (
    SELECT 
        EmployeeID,
        Name,
        ManagerID,
        DepartmentID,
        RoleID
    FROM Employees
    WHERE EmployeeID = 1
    
    UNION ALL
    
    SELECT 
        e.EmployeeID,
        e.Name,
        e.ManagerID,
        e.DepartmentID,
        e.RoleID
    FROM Employees e
    JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT 
    eh.EmployeeID,
    eh.Name AS EmployeeName,
    eh.ManagerID,
    d.DepartmentName,
    r.RoleName,
    GROUP_CONCAT(DISTINCT p.ProjectName ORDER BY p.ProjectName SEPARATOR ', ') AS ProjectNames,
    GROUP_CONCAT(DISTINCT t.TaskName ORDER BY t.TaskName SEPARATOR ', ') AS TaskNames
FROM EmployeeHierarchy eh
LEFT JOIN Departments d ON eh.DepartmentID = d.DepartmentID
LEFT JOIN Roles r ON eh.RoleID = r.RoleID
LEFT JOIN Projects p ON d.DepartmentID = p.DepartmentID
LEFT JOIN Tasks t ON eh.EmployeeID = t.AssignedTo
GROUP BY eh.EmployeeID, eh.Name, eh.ManagerID, d.DepartmentName, r.RoleName
ORDER BY eh.Name;
```
# Задача 12
```sql
WITH RECURSIVE EmployeeHierarchy AS (
    SELECT 
        EmployeeID,
        Name,
        ManagerID,
        DepartmentID,
        RoleID
    FROM Employees
    WHERE EmployeeID = 1
    
    UNION ALL
    
    SELECT 
        e.EmployeeID,
        e.Name,
        e.ManagerID,
        e.DepartmentID,
        e.RoleID
    FROM Employees e
    JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT 
    eh.EmployeeID,
    eh.Name AS EmployeeName,
    eh.ManagerID,
    d.DepartmentName,
    r.RoleName,
    GROUP_CONCAT(DISTINCT p.ProjectName ORDER BY p.ProjectName SEPARATOR ', ') AS ProjectNames,
    GROUP_CONCAT(DISTINCT t.TaskName ORDER BY t.TaskName SEPARATOR ', ') AS TaskNames,
    COUNT(DISTINCT t.TaskID) AS TotalTasks,
    COUNT(DISTINCT sub.EmployeeID) AS TotalSubordinates
FROM EmployeeHierarchy eh
LEFT JOIN Departments d ON eh.DepartmentID = d.DepartmentID
LEFT JOIN Roles r ON eh.RoleID = r.RoleID
LEFT JOIN Projects p ON d.DepartmentID = p.DepartmentID
LEFT JOIN Tasks t ON eh.EmployeeID = t.AssignedTo
LEFT JOIN Employees sub ON sub.ManagerID = eh.EmployeeID
GROUP BY eh.EmployeeID, eh.Name, eh.ManagerID, d.DepartmentName, r.RoleName
ORDER BY eh.Name;
```
# Задача 13

```sql
WITH RECURSIVE SubordinatesTree AS (
    SELECT 
        e.EmployeeID AS ManagerID,
        e.EmployeeID AS SubordinateID,
        1 AS Level
    FROM Employees e
    WHERE e.RoleID = (SELECT RoleID FROM Roles WHERE RoleName = 'Менеджер')
    
    UNION ALL
    
    SELECT 
        st.ManagerID,
        e.EmployeeID AS SubordinateID,
        st.Level + 1
    FROM SubordinatesTree st
    JOIN Employees e ON e.ManagerID = st.SubordinateID
),
ManagersWithSubordinates AS (
    SELECT 
        ManagerID,
        COUNT(DISTINCT SubordinateID) - 1 AS TotalSubordinates
    FROM SubordinatesTree
    GROUP BY ManagerID
    HAVING COUNT(DISTINCT SubordinateID) > 1 
)
SELECT 
    e.EmployeeID,
    e.Name AS EmployeeName,
    e.ManagerID,
    d.DepartmentName,
    r.RoleName,
    GROUP_CONCAT(DISTINCT p.ProjectName ORDER BY p.ProjectName SEPARATOR ', ') AS ProjectNames,
    GROUP_CONCAT(DISTINCT t.TaskName ORDER BY t.TaskName SEPARATOR ', ') AS TaskNames,
    mws.TotalSubordinates
FROM Employees e
JOIN ManagersWithSubordinates mws ON e.EmployeeID = mws.ManagerID
LEFT JOIN Departments d ON e.DepartmentID = d.DepartmentID
LEFT JOIN Roles r ON e.RoleID = r.RoleID
LEFT JOIN Projects p ON d.DepartmentID = p.DepartmentID
LEFT JOIN Tasks t ON e.EmployeeID = t.AssignedTo
GROUP BY e.EmployeeID, e.Name, e.ManagerID, d.DepartmentName, r.RoleName, mws.TotalSubordinates
ORDER BY e.Name;
```


