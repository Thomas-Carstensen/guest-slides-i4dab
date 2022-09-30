<!-- .slide: data-background="#003d73" -->

## SQL + DML

![SQL Connection](./img/sql-connection.jpg "SQL Connection")

![AU Logo](./../img/aulogo_uk_var2_white.png "AU Logo") <!-- .element style="width: 200px; position: fixed; bottom: 50px; left: 50px" -->


----

### Agenda

* Map relationships
* SQL Connection
* Domain Manipulation language

---

### Primary keys

* Primary key inline or as constraint

```sql
CREATE TABLE Movies (
    title varchar(50) PRIMARY KEY,
    year varchar(4), studio varchar(50),
    genre varchar(40)
)
-- or
CREATE TABLE Movies (
    title varchar(50), year varchar(4),
    studio varchar(50), genre varchar(40),
    CONSTRAINT pk_title PRIMARY KEY (title)
)

CREATE TABLE Movies (
  title varchar(50), year varchar(4),
  studio varchar(50), genre varchar(40),
  CONSTRAINT pk_title_year 
    PRIMARY KEY (title, year)
)
```
<!-- .element: style="font-size: 16px" -->

----

### Map 1-1 relations ships

* Interest defines where key is located

```sql
CREATE TABLE Car (
    reg VARCHAR(20) PRIMARY KEY,
    color VARCHAR(7),
)

CREATE TABLE ParkingSpot (
    id INT PRIMARY KEY,
    address VARCHAR(80),
    car_reg VARCHAR(20) UNIQUE
      FOREIGN KEY REFERENCES Car(reg),
)
```

----

### Map 1-N relations ships

* Set the foreign key constraint in _the_ many entity

```sql
CREATE TABLE Car (
    reg VARCHAR(20) PRIMARY KEY,
    color VARCHAR(7),
)

CREATE TABLE Wheels(
    id INT PRIMARY KEY,
    placement VARCHAR(2),
    car_reg VARCHAR(20)
      FOREIGN KEY REFERENCES Car(reg)
)
```

----

### Map N-N relations cars (1/2)

* In CarDriver `driver_cpr`/`car_reg` pair
* Optional: define pair as unique if ??

```sql
CREATE TABLE Car (
    reg VARCHAR(20) PRIMARY KEY,
    color VARCHAR(7),
)

CREATE TABLE Drivers(
    cpr INT PRIMARY KEY,
    Age int, Name VARCHAR(50)
)

CREATE TABLE CarDriver(
    driver_cpr INT FOREIGN KEY REFERENCES Drivers(cpr),
    car_reg VARCHAR(20) FOREIGN KEY REFERENCES Car(reg)
)
```
<!-- .element: style="font-size: 22px" -->

----

### Map N-N relations cars (2/2)

```sql
-- Or if the driver/car couple should be unique
CREATE TABLE CarDriver(
    driver_cpr INT FOREIGN KEY REFERENCES Drivers(cpr),
    car_reg VARCHAR(20) FOREIGN KEY REFERENCES Car(reg),
    CONSTRAINT uc_cpr_reg UNIQUE (driver_cpr, car_reg)
)

-- To create relations insert into CarDriver
INSERT INTO CAR VALUES ('12345', '123145'), ('123145', '21sdaf')
INSERT INTO Drivers VALUES (123456, 12, 'Alice'), 
    (43535, 18, 'Bob')
INSERT INTO CarDriver VALUES (123456,'12345'), (43535, '123145'), 
    (123456,'123145')
```
<!-- .element: style="font-size: 22px" -->


----

### Map inheritance

* Either in a single table
* or as a 1-1 relation with required existence

```sql
CREATE TABLE Products (
    id INT PRIMARY KEY,
    price INT, soup_type VARCHAR(10),
    expiration DATE, bakedOn VARCHAR(120),
    slices int, type VARCHAR(5)
)

INSERT INTO Products (id, price, soup_type, expiration, type)
VALUES (1, 10, 'mushroom', getDate(), 'soup'),
        (2, 15, 'shrimp', getDate(), 'soup')

INSERT INTO Products (id, price, bakedOn, slices, type)
VALUES (3, 9, 'full', 30, 'bread'),
       (4, 12, 'white', 25, 'bread')
```

---

### SQL Connection

* How to install
    * .Net Core 3.0+ (We'll be using 6.0)

```bash
$ mkdir SqlConnectionExample; cd SqlConnectionExample
$ dotnet new console
$ dotnet add package Microsoft.Data.SqlClient
## Or via package manager PM> Install-Package Microsoft.Data.SqlClient
$ dotnet run
```

----

### SqlConnection

```csharp
private static void CreateCommand(
    string queryString,
    string connectionString)
{
    using (SqlConnection connection
      = new SqlConnection(connectionString))
    {
        SqlCommand command = new SqlCommand(
             queryString, connection);
        command.Connection.Open();
        command.ExecuteNonQuery();
    }
}
```


----

### SQL Commands

```csharp
private static void ReadOrderData(
                    string connectionString) {
  string queryString = 
      "SELECT OrderID, CustomerID
         FROM dbo.Orders;";
  using (SqlConnection connection =
       new SqlConnection(
             connectionString)) {
    SqlCommand command = new SqlCommand(
        queryString, connection);
    connection.Open();
    using(SqlDataReader reader =
                 command.ExecuteReader()) {
      while (reader.Read()) {
         Console.WriteLine(String.Format("{0}, {1}",
          reader[0], reader[1]));
 } } } }
```
<!-- .element: style="font-size: 18px" -->

---

### Create, Read, Update & Delete

![Math is fun](./img/math.jpg "Math is fun")

----

### INSERT documentation

```sql
INSERT   
{  
        [ TOP ( expression ) [ PERCENT ] ]   
        [ INTO ]   
        { <object> | rowset_function_limited   
          [ WITH ( <Table_Hint_Limited> [ ...n ] ) ]  
        }  
    {  
        [ ( column_list ) ]   
        [ <OUTPUT Clause> ]  
        { VALUES ( { DEFAULT | NULL | expression } [ ,...n ] ) [ ,...n     ]   
        | derived_table   
        | execute_statement  
        | <dml_table_source>  
        | DEFAULT VALUES   
        }  
    }  
}  
```
<!-- .element: style="font-size: 20px" -->

----

### SQL syntax

* UPPERCASE - keyword
* Italic - arguments
* Bold - names (table, database ...)
    * On slides bold is focus
* `[,...n]`, `[...n]` - Item can be repeated (with comma or without)
* [] - optional
* {} - required
* `<block>` - block syntax

----

### Insert

* TOP (expression) [ PERCENT ] 
    * Specifies the number or percent of random rows that will be inserted. expression can be either a number or a percent of the rows. For more information, see TOP (Transact-SQL).

* Please read documentation before using :)


----

### SELECT documentation

```sql
<SELECT statement> ::=    
    [ WITH { [ XMLNAMESPACES ,] [ <common_table_expression> [,...n] ] } ]  
    <query_expression>   
    [ ORDER BY <order_by_expression> ] 
    [ <FOR Clause>]   
    [ OPTION ( <query_hint> [ ,...n ] ) ]
```

----

### Query specification

```sql
<query_specification> ::=   
SELECT [ ALL | DISTINCT ]   
    [TOP ( expression ) [PERCENT] [ WITH TIES ] ]   
    < select_list >   
    [ INTO new_table ]   
    [ FROM { <table_source> } [ ,...n ] ]   
    [ WHERE <search_condition> ]   
    [ <GROUP BY> ]   
    [ HAVING < search_condition > ]
```

----

## RELAX <!-- .element: style="color: white;" -->

<!-- .slide: data-background-image="./img/breathe.webp" -->

---

### Delete

```sql
DELETE   
    [ TOP ( expression ) [ PERCENT ] ]   
    [ FROM ]   
    { { table_alias  
      | <object>   
      | rowset_function_limited   
      [ WITH ( table_hint_limited [ ...n ] ) ] }   
      | @table_variable  
    }  
    [ <OUTPUT Clause> ]  
    [ FROM table_source [ ,...n ] ]   
    [ WHERE { <search_condition>   
            | { [ CURRENT OF   
                   { { [ GLOBAL ] cursor_name }   
                       | cursor_variable_name }   
                ]  
              }  }   
    ]   
    [ OPTION ( <Query Hint> [ ,...n ] ) ]  
```
<!-- .element: style="font-size: 17px" -->


----

### search_condition

```sql
<search_condition> ::=  
    MATCH(<graph_search_pattern>)
     | <search_condition_without_match>
     | <search_condition> AND <search_condition>

<search_condition_without_match> ::= 
    { [ NOT ] <predicate>
      | ( <search_condition_without_match> ) }   
    [ { AND | OR } [ NOT ] { <predicate>
             | ( <search_condition_without_match> ) } ]   
[ ...n ]   
```

----

#### Predicates

```sql
<predicate> ::=   
    { expression { = | < > | ! = | > | > = | ! > | < | < = | ! < } expression   
    | string_expression [ NOT ] LIKE string_expression   
  [ ESCAPE 'escape_character' ]   
    | expression [ NOT ] BETWEEN expression AND expression   
    | expression IS [ NOT ] NULL   
    | CONTAINS   
  ( { column | * } , '<contains_search_condition>' )   
    | FREETEXT ( { column | * } , 'freetext_string' )   
    | expression [ NOT ] IN ( subquery | expression [ ,...n ] )   
    | expression { = | < > | ! = | > | > = | ! > | < | < = | ! < }   
  { ALL | SOME | ANY} ( subquery )   
    | EXISTS ( subquery )     }  
```


----

### Update

```sql
UPDATE   
    [ TOP ( expression ) [ PERCENT ] ]   
    { { table_alias | <object> | rowset_function_limited   
         [ WITH ( <Table_Hint_Limited> [ ...n ] ) ]  
      }  
      | @table_variable      
    }  
    SET  
        { column_name = { expression | DEFAULT | NULL }  
          | { udt_column_name.{ { property_name = expression  
                                | field_name = expression }  
                                | method_name ( argument [ ,...n ] )  
                              }  
          }  
          | column_name { .WRITE ( expression , @Offset , @Length ) }  
          | @variable = expression  
          | @variable = column = expression  
          | column_name { += | -= | *= | /= | %= | &= | ^= | |= } expression  
          | @variable { += | -= | *= | /= | %= | &= | ^= | |= } expression  
          | @variable = column { += | -= | *= | /= | %= | &= | ^= | |= } expression  
        } [ ,...n ]   
  
    [ <OUTPUT Clause> ]  
    [ FROM{ <table_source> } [ ,...n ] ]   
    [ WHERE { <search_condition>   
            | { [ CURRENT OF   
                  { { [ GLOBAL ] cursor_name }   
                      | cursor_variable_name   
                  }   
                ]  
              }  
            }   
    ]   
    [ OPTION ( <query_hint> [ ,...n ] ) ]  
[ ; ] 
```
<!-- .element: style="font-size: 14px" -->

---

![Exercises](./img/exercise.jpg "Exercises")

----

### References

* Meme 1: https://rtask.thinkr.fr/the-ten-commandments-for-a-well-formatted-database/

* Gif: https://giphy.com/gifs/13HgwGsXF0aiGY/media