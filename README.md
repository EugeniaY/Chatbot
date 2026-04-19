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

7. Open n8n (using node js)
   - Win + R
   - cmd
   - npx n8n
   - Type "o" to open browser

8. Connect Postgress to n8n
   - Create workflow
   - Click "+" Add first step..
   - Find and Click Postgres
   - Click Execute a SQL query
   - Credential to connect with -> Create new credential -> Fill in the credential like this
     ```sql
      Host: localhost
      Database: auroratech_db
      User: postgres
      Password: your postgres password
      Port: 5432
      SSL: Disable
      ```
   - Click Save
   - Query:
     ```sql
      SELECT * FROM users LIMIT 5;
      ```
   - Click Execute Step
  
   9. Login System
      - Replace the cursor icon with Webhook
        <img width="1708" height="800" alt="image" src="https://github.com/user-attachments/assets/23e08c15-ea18-4eb4-bb40-0ccd51f1c68d" />

      - HTTP Method: POST
      - Path: login
      - Authentication: None
      - Respond: Immediately
      - Click X to close
      - Double click the "Execute a SQL query"
      - Change the Query to
        ```sql
         SELECT id,name,username,department,email
         FROM users
         WHERE username='{{ $json.body.username }}'
         AND password='{{ $json.body.password }}';
         ```
      - Click X to close
      - Double click the Webhook
      - Click Listen for test event
      - Open cmd
      - Paste
        ```
         curl -X POST http://localhost:5678/webhook-test/login ^
         -H "Content-Type: application/json" ^
         -d "{\"username\":\"peter1\",\"password\":\"RVMRx1vYgi\"}"
         ```
      - Result in Webhook will shows like this
        <img width="800" height="1340" alt="image" src="https://github.com/user-attachments/assets/d008fb68-0b07-44d4-8483-d9232f559d53" />
      -  Double click the "Execute a SQL query"
      -  Execute workflow
     
   10. Add If
       - Click the "+"
       - Search and click If
       - Click value 1
          - {{ $items().length }}
          - Click '\/' to number -> is greater than
       - Click value2 and fill in with 0
       - Click Execute step
         <img width="800" height="1158" alt="image" src="https://github.com/user-attachments/assets/4402cdee-5421-419b-8378-e0e85beb520c" />

   11. Respond to Webhook
       - Click true "+"
       - Search and Click Respond to Webhook
       - Change Respond With: JSON
       - Fill the respond body (Expression)
         ```
         {{ {
           status: "success",
           message: "Login successful",
           user: {
             id: $json.id,
             name: $json.name,
             username: $json.username,
             department: $json.department,
             email: $json.email
           }
         } }}
         ```
         - Click Execute step
         <img width="2797" height="1360" alt="image" src="https://github.com/user-attachments/assets/230a3256-07b0-4420-8dc5-79b35da1c36b" />




  

