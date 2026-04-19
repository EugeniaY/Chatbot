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
        
        <img width="2765" height="1352" alt="image" src="https://github.com/user-attachments/assets/1d052790-017b-4d22-b855-d4485e27d8d9" />

      -  Double click the "Execute a SQL query"
      -  Execute workflow
     
   10. Add If
       - Click the "+"
       - Search and click If
       - Click value 1
          - {{ $json.id }}
          - Click '\/' to number -> exists
       - Click value2 and fill in with 0
       - Click Execute step
         <img width="2765" height="1351" alt="image" src="https://github.com/user-attachments/assets/b6c60d59-cef0-400b-ae65-9caeb5ac8df7" />


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

  11b. Respond to Webhook (2)
       - Click false "+"
       - Search and Click Respond to Webhook
       - Change Respond With: JSON
       - Fill the respond body (Expression)
         ```
         {{ {
           status: "fail",
           message: "Invalid username or password"
         } }}
         ```
      - Click Execute step
      - Double Click Webhook (The very first one)
      - Change Respond: Using 'Respond to Webhook' Node

   12. Test if Works
       - Click Webhook 'Listen for test event'
       - Open cmd
       - Paste
          ```
         curl -X POST http://localhost:5678/webhook-test/login ^
         -H "Content-Type: application/json" ^
         -d "{\"username\":\"peter1\",\"password\":\"RVMRx1vYgi\"}"
         ```
       - Should Appear like this
         <img width="2345" height="380" alt="image" src="https://github.com/user-attachments/assets/2d4a1206-060b-4043-8f2f-f41b643cb995" />

   13. Rename to AuroraTech Login API and Publish the n8n
   14. Create Login Page using HTML, CSS      

## Employee List
1. Copy the previous Work flow and rename the workflow to AuroraTech Employees API
2. Double Click Webhook, change only below and keep the rest
   - Path: employees
   - HTTP Method: Get
3. Double Click "Execute a SQL query" change the Query to:
    ```sql
         SELECT id, name, username, department, email
         FROM users
         ORDER BY id;
    ```
    - Click Execute previous nodes
    - Delete IF
    - Click "+" and Search and Click Respond to Webhook
       - Respond With: All Incoming Items
       - Execute Step
4. Publish
5. Create employees.html

## Assistant AI chatbot
1. Copy the AuroraTech Login API
2. Double Click Webhook, change only below and keep the rest
   - Path: assistant
   - HTTP Method: POST
3. Double Click "Execute a SQL query" change the Query to:
    ```sql
         SELECT id, name, username, department, email
         FROM users
         WHERE LOWER(department) = LOWER('{{ $json.body.message }}')
         ORDER BY id;
    ```
    - Click Execute previous nodes
    - Delete IF
    - Click "+" and Search and Click Respond to Webhook
       - Respond With : JSON
       - Response Body (Expression) : 
         ```
         {{ $input.all().map(item => item.json) }}
         ```
       - Execute Step


  

