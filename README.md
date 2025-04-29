# SQL Server User Management in Docker

This guide provides common T-SQL commands to manage users and permissions in a SQL Server container.

---

## Connect to the SQL Server Container

Run the following command to connect using `sqlcmd`:

```bash
docker exec -it mssql-server-instance /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P 'YourStrong@PasswOrd' -C
```

Replace `'YourStrong@PasswOrd'` with your actual SA password.

---

### Create a new login (server-level principal)

```sql
CREATE LOGIN [username] WITH PASSWORD = 'UserPassword123!';
GO
```

---

### Create a database user mapped to the login in a specific database

```sql
USE [YourDatabase];
GO
CREATE USER [username] FOR LOGIN [username];
GO
```

---

### Grant roles/permissions to the user

For example, to make the user a database owner:

```sql
ALTER ROLE db_owner ADD MEMBER [username];
GO
```

Or to grant specific permissions:

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::dbo TO [username];
GO
```

---

### Remove a user from a database

```sql
USE [YourDatabase];
GO
DROP USER [username];
GO
```

---

### Remove a login from the server

```sql
DROP LOGIN [username];
GO
```

---

### Example: Create a user with read-only access

```sql
CREATE LOGIN [readonlyuser] WITH PASSWORD = 'ReadOnlyPass123!';
GO
USE [YourDatabase];
GO
CREATE USER [readonlyuser] FOR LOGIN [readonlyuser];
GO
ALTER ROLE db_datareader ADD MEMBER [readonlyuser];
GO
```

---

To query all available **logins** or **users** in SQL Server, you can use the following T-SQL commands. These can be executed using `sqlcmd` or any SQL client connected to your SQL Server instance.

---

### 1. **Query All Logins (Server-Level Principals)**

To list all logins on the SQL Server instance:

```sql
SELECT name, type_desc, is_disabled
FROM sys.server_principals
WHERE type IN ('S', 'U', 'G'); -- S = SQL login, U = Windows login, G = Windows group
GO
```

- `name`: The name of the login.
- `type_desc`: The type of login (e.g., SQL login, Windows login).
- `is_disabled`: Indicates whether the login is disabled (1 = disabled, 0 = enabled).

---

### 2. **Query All Users in a Specific Database**

To list all users in a specific database:

```sql
USE [YourDatabase];
GO
SELECT name, type_desc, create_date, modify_date
FROM sys.database_principals
WHERE type IN ('S', 'U', 'G', 'A', 'R'); -- S = SQL user, U = Windows user, G = Windows group, A = Application role, R = Database role
GO
```

- `name`: The name of the user.
- `type_desc`: The type of user (e.g., SQL user, Windows user, database role).
- `create_date`: The date the user was created.
- `modify_date`: The date the user was last modified.

---

### 3. **Query Permissions for a Specific User**

To view the permissions granted to a specific user in a database:

```sql
USE [YourDatabase];
GO
SELECT grantee_principal.name AS UserName,
       permission_name,
       state_desc AS PermissionState,
       class_desc AS PermissionClass,
       object_name(major_id) AS ObjectName
FROM sys.database_permissions
JOIN sys.database_principals AS grantee_principal
    ON sys.database_permissions.grantee_principal_id = grantee_principal.principal_id
WHERE grantee_principal.name = 'username';
GO
```

Replace `'username'` with the name of the user you want to query.
