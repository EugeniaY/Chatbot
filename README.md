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
   - Create assistant.html
  
## Update Dashboard Stats
1. Add node Code between Webhook and Execute a SQL query (Javascript)
   ```
         const raw = ($json.body?.message || "").toLowerCase().trim();
         const departments = [
           "marketing",
           "finance",
           "it",
           "admin",
           "sales",
           "hr",
           "operations"
         ];
         
         let matchedDepartment = null;
         
         for (const dept of departments) {
           if (raw.includes(dept)) {
             matchedDepartment = dept;
             break;
           }
         }
         
         return [
           {
             json: {
               originalMessage: raw,
               department: matchedDepartment
             }
           }
         ];
    ```
   - Execute Step
 2. Change the SQL node
    ```sql
         SELECT id, name, username, department, email
         FROM users
         WHERE LOWER(department) = LOWER('{{ $json.department }}')
         ORDER BY id;
    ```
3. Add If
   - Conditions: {{ $json.department }}
   - operator -> string : is not empty
   <img width="1690" height="518" alt="image" src="https://github.com/user-attachments/assets/b0ceeb2e-19a8-42b0-bc02-31ea4e2dff57" />

4. Add "Respond to Webhook" for false
   - Respond With: JSON
   - Response Body (expression):
     ```
         {{ {
           status: "fail",
           message: "Department not recognized. Try: Marketing, Finance, IT, Admin, Sales, HR, Operations."
         } }}
    ```
5. Execute Workflow
6. Open cmd
   - Check valid Input
   ```
         curl -X POST http://localhost:5678/webhook-test/assistant ^
         -H "Content-Type: application/json" ^
         -d "{\"message\":\"show marketing employees\"}"
    ```
   - Check invalid input
      ```
         curl -X POST http://localhost:5678/webhook-test/assistant ^
         -H "Content-Type: application/json" ^
         -d "{\"message\":\"show manager employees\"}"
       ```
    - should appear like:
      
      {
          "status":"fail",
          "message":"Department not recognized..."
      }

  ## AuroraTech Dashboard Stats API
  1. Copy AuroraTech Employee API workfloq
  2. Double Click Webhook
     - Method: GET
     - Path: dashboard-stats
     - Respond: Using 'Respond to Webhook' Node
3. Double Click "Execute a SQL query" change the Query to:
    ```sql
         SELECT
        (SELECT COUNT(*) FROM users) AS total_employees,
        (SELECT COUNT(DISTINCT department) FROM users) AS total_departments,
        (SELECT COUNT(*) FROM users WHERE LOWER(department) = 'marketing') AS marketing_count,
        (SELECT COUNT(*) FROM users WHERE LOWER(department) = 'finance') AS finance_count;
    ```
4. Double Click Respond to Webhook
   - Respond With: JSON
   - Response Body (Expression) : {{ $json }}
   
5. Test by opening http://localhost:5678/webhook/dashboard-stats
   - Must show like
     ```
         {
           "total_employees": 300,
           "total_departments": 7,
           "marketing_count": 54,
           "finance_count": 41
         }
    ```
7. Test the dashboard.html

## Update AI Assistant Feature
- show marketing employees
- list finance staff
- who works in IT
- give me admin employees
1. Go to AuroraTech Assistant API in n8n
2. Double Click Code and change like this
    ```
         const text = ($json.body?.message || "").toLowerCase().trim();
         const departments = [
           "marketing",
           "finance",
           "it",
           "admin",
           "sales",
           "hr",
           "operations"
         ];
         
         let department = null;
         
         for (const dep of departments) {
           if (text.includes(dep)) {
             department = dep;
             break;
           }
         }
         
         let intent = "unknown";
         
         if (department) {
           intent = "list";
         }
         
         if ((text.includes("how many") || text.includes("count")) && department) {
           intent = "count";
         }
         
         if (
           text.includes("top department") ||
           text.includes("most employees") ||
           text.includes("largest department")
         ) {
           intent = "top_department";
         }
         
         return [
         {
           json: {
             originalMessage: text,
             department,
             intent
           }
         }
         ];
    ```
    3. Double Click Execute a SQL query (IF -> true)
       - Rename SQL Count
       - Change the Query
         ```
         {
           "total_employees": 300,
           "total_departments": 7,
           "marketing_count": 54,
           "finance_count": 41
         }
       ```
   4. Add If in (IF -> False)
      - Rename Title "IF Top Department"
      - condition: {{ $json.department }}
      - operator (string): is not empty

      a. (IF -> False -> If -> True)
         - Add a Execute a SQL query node
         - Rename SQL Top Dept
         - Change the Query:
            ```sql
            SELECT department, COUNT(*) AS total
            FROM users
            GROUP BY department
            ORDER BY total DESC
            LIMIT 1;
            ```
      b. (IF -> False -> If -> False)
         - Add If
         - Rename the title "IF Department Exists"
         - Condition: {{ $json.department }}
         - operator (string): is not empty
              - (If -> True)
                    - Copy SQL Top Dept and connect with If -> True
                    - Rename SQL List
                    - Change the Query
                      ```sql
                           SELECT id,name,username,email,department
                           FROM users
                           WHERE LOWER(department)=LOWER('{{ $json.department }}');
                     ```
             - (IF -> False -> If -> False)
               - Connect the Respond to Webhook1 with IF -> False
               - the one that has
                 ```sql
                           {{ {
                           status:"fail",
                           message:"Department not recognized."
                           } }}
                 ```
                 
