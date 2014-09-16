# Joins

- SQLite supported joins
  
  - Cross Join
  ```sql
  SELECT *
  FROM employee CROSS JOIN department;
  ```

  - Inner Join
  ```sql
  SELECT * FROM TableA
  INNER JOIN TableB
  ON TableA.name = TableB.name;
  ```

  - Left Outer Join
  ```sql
  SELECT * FROM TableA
  LEFT OUTER JOIN TableB
  ON TableA.name = TableB.name;
  ```

- Other types of joins:
  - Full Outer Join
  - Right Outer Join
  - Jeff Atwoods Article


- Join tables
  - Creating a join table for wizard powers
    - wizards might share powers
    - powers might have different strength for different wizards
    - a table that links two tables
    - might store other information
  - Join table join via join/inner join

