# Soda Core Demo with MySQL

## Introduction to Soda Core

Soda Core is an open-source data reliability tool that helps data teams detect, analyze, and resolve data issues. Compared to other tools like Great Expectations, Soda Core is often appreciated for its simplicity and ease of use, especially for teams looking to implement data quality checks without extensive setup or configuration.

## Set Up

### Prerequisites

Before setting up Soda Core, make sure you have Python installed on your machine. It is recommended to use a virtual environment to manage dependencies.

### Creating a Virtual Environment

Run the following commands in your terminal to set up a Python virtual environment:

```bash
python -m venv soda-env
source soda-env/bin/activate
```

### Installing Soda core
With the virtual environment activated, install Soda Core for MySQL by running:
```bash
pip install soda-core-mysql
```
### Testing with Docker MySQL

You can run the following command to start a mysql container:
```bash
docker run --name mysql-container -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql:latest
```

### Set up the DB
Use a MySQL client to connect to the running MySQL instance (it will be available at localhost:3306). Run the following SQL commands to set up your database:

```sql
CREATE DATABASE demo_db;

CREATE TABLE demo_db.campaigns (
    campaign_id INT PRIMARY KEY,
    campaign_name VARCHAR(255) NOT NULL,
    clicks INT DEFAULT 0,
    spending FLOAT DEFAULT 0.0,
    impressions INT DEFAULT 0,
    date DATE
);

INSERT INTO demo_db.campaigns (campaign_id, campaign_name, clicks, spending, impressions, date) VALUES
(0, 'id_0', 150, 25.50, 1200, '2023-08-15'),
(1, 'id_1', 200, 32.00, 1850, '2023-08-10'),
(2, 'id_2', 80, 18.75, 900, '2023-08-20'),
(3, 'id_3', 135, 28.25, 1100, '2023-08-18'),
(4, 'id_4', 250, 45.00, 2200, '2023-08-12'),
(5, 'id_5', 110, 22.00, 1050, '2023-08-22'),
(6, 'id_6', 185, 35.75, 1700, '2023-08-16'),
(7, 'id_7', 95, 20.50, 880, '2023-08-19'),
(8, 'id_8', 210, 38.00, 1950, '2023-08-13'),
(9, 'id_9', 140, 26.75, 1150, '2023-08-21');
```

## Soda Configuration

To use Soda with your MySQL instance, you need to create some configuration files that Soda will use to connect to your database and to define the rules for your data quality checks.

### Data Source Configuration

Create a file named `datasource.yaml` in your project directory with the following content. This file configures the connection to your MySQL database.

```yaml
data_source simple_mysql:
  type: mysql
  connection:
    host: localhost
    port: '3306'
    username: root
    password: password
  database: demo_db
```
Create another file named *quality_check.yaml* in your project directory. This file specifies the data quality checks to be performed on your campaigns table. Here's how to set it up:
```yaml
checks for campaigns:
  - row_count > 0
  - avg(spending) > 10
  - duplicate_count(campaign_id) = 0
  - duplicate_count(campaign_name) = 0
  - count = 0:
      count query: |
        SELECT COUNT(*) FROM campaigns WHERE spending > 50 AND clicks < 100
  - freshness(date):
      warn: when > 5h
      fail: when > 2d
  - failed rows:
      samples limit: 5
      fail condition: impressions > 0 and spending = 0
  - invalid_count(clicks) = 0:
      valid format: positive integer
```

### Running the check
```bash
soda scan -d simple_mysql -c configuration.yaml quality_check.yaml --verbose -srf result.json
```