

# InfraLinker Application

This repository contains the InfraLinker application, a Flask-based app with a MySQL backend, designed to manage IT infrastructure data including departments, admins, device roles, and tags.

## Docker Compose Setup

This is the Docker Compose file used to run your application. It defines two services: the Flask app and a MySQL database.

```yaml
version: '3.8'

services:
  app:
    image: oubenssaidyoussef/infralinker:v1.0
    ports:
      - "5000:5000"
    environment:
      - FLASK_CONFIG=development
      - MYSQL_USER=youssef
      - MYSQL_PASSWORD=youssef
      - MYSQL_HOST=db
    depends_on:
      - db

  db:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=rootpassword
      - MYSQL_DATABASE=infralinker
      - MYSQL_USER=youssef
      - MYSQL_PASSWORD=youssef
    volumes:
      - mysql_data:/var/lib/mysql
    command: --default-authentication-plugin=mysql_native_password

volumes:
  mysql_data:
```

## Running the Application

1. Start the containers in detached mode:

   ```bash
   docker-compose up -d
   ```

2. Initialize and migrate the database (run these commands only once):

   ```bash
   docker-compose exec app flask db init
   docker-compose exec app flask db migrate -m "Initial migration"
   docker-compose exec app flask db upgrade
   ```

## Database Initialization SQL

After setting up the app, you can initialize your database with the following SQL commands:

```sql
USE infralinker;

-- Insert initial data into departments table
INSERT IGNORE INTO departments (id, name, description) VALUES 
(1, 'IT', 'IT Infrastructure Department');

-- Insert initial admin user data
INSERT IGNORE INTO admins (
    id, firstname, lastname, email, phone, function, password_hash, 
    is_admin, is_manager, control_racks, control_platforms, control_networks, 
    control_servers, control_applications, department_id, change_password, last_seen
) VALUES (
    1, 'Admin', 'Administrator', 'admin@infralinker.com', '060-000-0006', 'Application Administrator', 
    'pbkdf2:sha256:150000$5l5suuHR$b63b9521dbcf42e9a89cea7d03a51ae79fe3748f381f0dc24ffc11fd8b3b7b2b', 
    1, 1, 1, 1, 1, 1, 0, 1, NOW()
);

-- Insert tags
INSERT IGNORE INTO tags (id, tag_name, tag_description, tag_color, add_by, add_date) VALUES 
(1, 'PROD', 'PRODUCTION ENVIRONMENT', '#0B8C5A', 1, '2020-12-31'),
(2, 'TEST', 'TEST ENVIRONMENT', '#F252D2', 1, '2020-12-31'),
(3, 'DMZ', 'DEMILITARIZED ZONE', '#CCF3FF', 1, '2020-12-31'),
(4, 'PRIV', 'PRIVATE ZONE', '#DD6E88', 1, '2020-12-31'),
(5, 'PUBL', 'PUBLIC ZONE', '#4B0170', 1, '2020-12-31');

-- Insert device roles
INSERT IGNORE INTO device_roles (id, name, description, device_color) VALUES
(1, 'SW SAN', 'SWITCH SAN', '#e5bb90'),
(2, 'SW TOR', 'TOR SWITCH', '#079653'),
(3, 'SW LAN', 'SWITCH LAN', '#04c994'),
(4, 'NAS', 'Network Attached Storage', '#ef4750'),
(5, 'LB', 'LOAD BALANCER', '#a088f7'),
(6, 'FW PALO', 'FIREWALL PALO ALT', '#9eef97'),
(7, 'INTEL', 'SERVER INTEL', '#9fbae0'),
(8, 'POWER', 'SERVER POWER', '#71bf39'),
(9, 'FW ASA', 'FIREWALL CISCO ASA', '#b19ff2'),
(10, 'ROUTER', 'ROUTER', '#4c6bad');

-- Create view platforms_warranty
CREATE OR REPLACE VIEW platforms_warranty AS
SELECT id, platform_name, end_warranty_date, 'OUT' AS status
FROM platforms 
WHERE DATE(end_warranty_date) < CURDATE()
UNION ALL
SELECT id, platform_name, end_warranty_date, 'IN' AS status
FROM platforms 
WHERE DATE(end_warranty_date) >= CURDATE();

-- Create view contract_validity
CREATE OR REPLACE VIEW contract_validity AS
SELECT id, contract_number, start_date, end_date, 'EXPIRED' AS status
FROM contracts 
WHERE DATE(end_date) < CURDATE()
UNION ALL
SELECT id, contract_number, start_date, end_date, 'VALID' AS status
FROM contracts 
WHERE DATE(end_date) >= CURDATE();
```

> Note: To exit MySQL CLI, use `exit` in your terminal/shell, not inside the SQL script.

## Summary

- Use `docker-compose up -d` to start the app and database.
- Use the flask db commands inside the app container for migrations.
- Initialize your database tables and views with the SQL provided above.

If you have any questions or issues, please open an issue or contact the maintainer.

---
## Authentication to application
https://server_ip_adresse login: admin@infralinker.com password: ROOT1234
