# Chatbot
Designing chatbot

## Connect Local PostgreSQL
1. Add new Server
   
  Tab 1: General

  - Name : Local PostgreSQL

Tab 2: Connection
  - Host name/address: localhost
  - Port: 5432
  - Maintenance database: postgres
  - Username: postgres
  - Password: (password when you install PostgreSQL)
  - ✔ tick Save Password
    
Then click Save

2. Create Database Project
   - Databases → Right Click → Create → Database
   - Database: auroratech_db

Then click Save

3. Open Query Tool
   - Tools → Query Tool

4. Create table users
```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  username VARCHAR(50) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  department VARCHAR(50),
  email VARCHAR(100) UNIQUE
);
```

Then click Excute

5. Import file CSV
   - auroratech_db -> Schemas -> public -> Tables -> users -> right click -> Import/Export Data -> click folder -> choose your csv file
   - Format: csv
   - Encoding: UTF8
   - Click Tab Options-> Header -> Turn ON

   Click OK

6. Test Query
   - aurora_db -> Tools -> Query Tool

```sql
   SELECT * FROM users LIMIT 10;
```

Click Execute
   
