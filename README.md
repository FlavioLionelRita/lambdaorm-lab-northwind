# Lab Northwind

In this laboratory we will see:

Creating the northwind sample database tables and loading it with sample data.
This database presents several non-standard cases such as:
	- Name of tables and fields with spaces
	- Tables with composite primary keys
	- Tables with autonumeric ids and others with ids strings

Since this is the database that was used for many examples and unit tests, you can test the example queries that are in the documentation.
We will also see some example queries to execute from CLI

## Pre requirements

### Create database for test

Create file "docker-compose.yaml"

```yaml
version: '3'
services:
  test:
    container_name: lambdaorm-test
    image: mysql:5.7
    restart: always
    environment:
      - MYSQL_DATABASE=lab_northwind
      - MYSQL_USER=test
      - MYSQL_PASSWORD=test
      - MYSQL_ROOT_PASSWORD=root
    ports:
      - 3306:3306
```

Create MySql database for test:

```sh
docker-compose up -d
```

Create user and set character:

```sh
docker exec lambdaorm-test  mysql --host 127.0.0.1 --port 3306 -uroot -proot -e "ALTER DATABASE lab_northwind CHARACTER SET utf8 COLLATE utf8_general_ci;"
docker exec lambdaorm-test  mysql --host 127.0.0.1 --port 3306 -uroot -proot -e "GRANT ALL ON *.* TO 'test'@'%' with grant option; FLUSH PRIVILEGES;"
```

### Install lambda ORM CLI

Install the package globally to use the CLI commands to help you create and maintain projects

```sh
npm install lambdaorm-cli -g
```

## Test

### Create project

will create the project folder with the basic structure.

```sh
lambdaorm init -w lab_northwind
```

position inside the project folder.

```sh
cd lab_northwind
```

### Complete Schema

In the creation of the project the schema was created but without any entity.
Add the Country entity as seen in the following example

```yaml
app:
  src: src
  data: data
  models: models
  defaultDatabase: mydb
databases:
  - name: mydb
    dialect: mysql
    schema: northwind
    connection:
      host: localhost
      port: 3306
      user: test
      password: test
      database: lab_northwind
      multipleStatements: true
      waitForConnections: true
      connectionLimit: 10
      queueLimit: 0
schemas:
  - name: northwind
    entities:
      - name: Address
        abstract: true
        properties:
          - name: address
            mapping: Address
          - name: city
            mapping: City
          - name: region
            mapping: Region
          - name: postalCode
            mapping: PostalCode
            length: 20
          - name: country
            mapping: Country
        indexes:
          - name: postalCode
            fields: ["postalCode"]
          - name: region
            fields: ["region", "country"]
          - name: city
            fields: ["city"]
      - name: Categories
        mapping: Categories
        primaryKey: ["id"]
        uniqueKey: ["name"]
        properties:
          - name: id
            mapping: CategoryID
            type: integer
            nullable: false
            autoincrement: true
          - name: name
            mapping: CategoryName
            nullable: false
          - name: description
            mapping: Description
            length: 1000
      - name: Customers
        extends: Address
        mapping: Customers
        primaryKey: ["id"]
        indexes:
          - name: name
            fields: ["name"]
        properties:
          - name: id
            mapping: CustomerID
            length: 5
            nullable: false
          - name: name
            mapping: CompanyName
            nullable: false
          - name: contact
            mapping: ContactName
          - name: phone
            mapping: ContactTitle
        relations:
          - name: orders
            type: manyToOne
            composite: true
            from: id
            entity: Orders
            to: customerId
      - name: Employees
        extends: Address
        mapping: Employees
        primaryKey: ["id"]
        uniqueKey: ["lastName", "firstName"]
        properties:
          - name: id
            mapping: EmployeeID
            type: integer
            nullable: false
            autoincrement: true
          - name: lastName
            mapping: LastName
            nullable: false
          - name: firstName
            mapping: FirstName
            nullable: false
          - name: title
            mapping: Title
          - name: titleOfCourtesy
            mapping: TitleOfCourtesy
          - name: birthDate
            mapping: BirthDate
            type: datetime
          - name: hireDate
            mapping: HireDate
            type: datetime
          - name: phone
            mapping: HomePhone
          - name: reportsToId
            mapping: ReportsTo
            type: integer
        relations:
          - name: reportsTo
            from: reportsToId
            entity: Employees
            to: id
      - name: Shippers
        mapping: Shippers
        primaryKey: ["id"]
        uniqueKey: ["name"]
        properties:
          - name: id
            mapping: ShipperID
            type: integer
            nullable: false
            autoincrement: true
          - name: name
            mapping: CompanyName
            nullable: false
          - name: phone
            mapping: Phone
            length: 20
      - name: Suppliers
        extends: Address
        mapping: Suppliers
        primaryKey: ["id"]
        uniqueKey: ["name"]
        properties:
          - name: id
            mapping: SupplierID
            type: integer
            nullable: false
            autoincrement: true
          - name: name
            mapping: CompanyName
            nullable: false
          - name: contact
            mapping: ContactName
          - name: phone
            mapping: Phone
            length: 20
          - name: homepage
            mapping: HomePage
            length: 200
      - name: Products
        mapping: Products
        primaryKey: ["id"]
        uniqueKey: ["name", "supplierId"]
        properties:
          - name: id
            mapping: ProductID
            type: integer
            nullable: false
            autoincrement: true
          - name: name
            mapping: ProductName
            nullable: false
          - name: supplierId
            mapping: SupplierID
            nullable: false
            type: integer
          - name: categoryId
            mapping: CategoryID
            type: integer
          - name: quantity
            mapping: QuantityPerUnit
          - name: price
            mapping: UnitPrice
            type: decimal
            default: 0
          - name: inStock
            mapping: UnitsInStock
            type: decimal
            default: 0
          - name: onOrder
            mapping: UnitsOnOrder
            type: decimal
            default: 0
          - name: reorderLevel
            mapping: ReorderLevel
            type: decimal
            default: 0
          - name: discontinued
            mapping: Discontinued
            type: boolean
            default: false
        relations:
          - name: supplier
            from: supplierId
            entity: Suppliers
            to: id
          - name: category
            from: categoryId
            entity: Categories
            to: id
      - name: Orders
        mapping: Orders
        primaryKey: ["id"]
        indexes:
          - name: orderDate
            fields: ["orderDate"]
          - name: shippedDate
            fields: ["shippedDate"]
        properties:
          - name: id
            mapping: OrderID
            type: integer
            nullable: false
            autoincrement: true
          - name: customerId
            mapping: CustomerID
            nullable: false
            length: 5
          - name: employeeId
            mapping: EmployeeID
            nullable: false
            type: integer
          - name: orderDate
            mapping: OrderDate
            type: datetime
          - name: requiredDate
            mapping: RequiredDate
            type: datetime
          - name: shippedDate
            mapping: ShippedDate
            type: datetime
          - name: shipViaId
            mapping: ShipVia
            type: integer
          - name: freight
            mapping: Freight
            type: decimal
          - name: name
            mapping: ShipName
          - name: address
            mapping: ShipAddress
          - name: city
            mapping: ShipCity
          - name: region
            mapping: ShipRegion
          - name: postalCode
            mapping: ShipPostalCode
            length: 20
          - name: country
            mapping: ShipCountry
        relations:
          - name: customer
            from: customerId
            entity: Customers
            to: id
          - name: employee
            from: employeeId
            entity: Employees
            to: id
          - name: details
            type: manyToOne
            composite: true
            from: id
            entity: OrderDetails
            to: orderId
      - name: OrderDetails
```

### Update

```sh
lambdaorm update
```

the file model.ts will be created inside src/models/countries.

### Sync

```sh
lambdaorm sync
```

It will generate:

- the tables will be created in the database and a status file "mydb-state.json" in the "data" folder.

### Popuplate Data

then we execute

```sh
lambdaorm import -s ./data.json -n mydb
```

### Queries

Shows some fields of the first product:

```sh
lambdaorm run -e "Products.first(p => ({ category: p.category.name, name: p.name, quantity: p.quantity, inStock: p.inStock }))"
```

lists details of orders that meet a filter and sorts the records
the values to filter are passed as parameters:

```sh
lambdaorm run -e "OrderDetails.filter(p => between(p.order.shippedDate, from, to) && p.unitPrice > minValue).map(p => ({ category: p.product.category.name, product: p.product.name, unitPrice: p.unitPrice, quantity: p.quantity })).sort(p => [p.category, p.product])" -d '{ ""minValue"": 10, ""from"": ""1997-01-01"", ""to"": ""1997-12-31"" }'
```

List the maximum price by category, ordered by descending price and filtering by maximum price greater than 100

```sh
lambdaorm run -e "Products.having(p => max(p.price) > 100).map(p => ({ category: p.category.name, largestPrice: max(p.price) })).sort(p => desc(p.largestPrice))"
```

distinct category of products:

```sh
lambdaorm run -e "Products.distinct(p => ({ quantity: p.quantity, category: p.category.name })).sort(p => p.category)"
```

returns an order including customer fields, order detail, product and category:

```sh
lambdaorm run -e "Orders.filter(p => p.id == id).include(p => [p.customer.map(p => p.name), p.details.include(p => p.product.include(p => p.category.map(p => p.name)).map(p => p.name)).map(p => [p.quantity, p.unitPrice])])" -d '{""id"": 1}'
```

### Drop

remove all tables from the schema and delete the state file, mydb-state.json

```sh
lambdaorm drop -n mydb
```

## End

### Remove database for test

Remove MySql database:

```sh
docker-compose down
```
