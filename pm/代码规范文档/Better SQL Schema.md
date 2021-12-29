# Better SQL Schema

There are a lot of decisions to make when creating new tables and data warehouses. Some that seem inconsequential at the time end up causing you and your users pain for the life of the database.

We’ve worked with thousands of people and their databases and, after countless hours of reading and writing queries, we’ve seen almost everything. Here are our top 10 rules for creating pain-free schemas.

### 1. Only Use Lowercase Letters, Numbers, and Underscores

Don’t use dots, spaces, or dashes in database, schema, table, or column names. Dots are for identifying objects, usually in the *database.schema.table.column* pattern.

Having dots in names of objects will cause confusion. Likewise, using spaces in object names will force you to add a bunch of otherwise unnecessary quotes to your query:

> **select** *"user name"* **from** events

> *-- vs*

> **select** user_name **from** events

Queries are harder to write if you use capital letters in table or column names. If everything is lowercase, no one has to remember if the **users** table is **Users** or **users**.

And when you eventually change databases or replicate your tables into a warehouse, you won’t need to remember which database is case-sensitive, as only some are.

### 2. Use Simple, Descriptive Column Names

If the **users** table needs a foreign key to the *packages* table, name the key *package_id*. Avoid short and cryptic names like *pkg_fk*; others won’t know what that means. Descriptive names make it easier for others to understand the schema, which is vital to maintaining efficiency as the team grows.

Don’t use ambiguous names for [polymorphic data](http://en.wikipedia.org/wiki/Polymorphism_(computer_science)). If you find yourself creating columns with an *item_type* or *item_value* pattern, you’re likely better off using more columns with specific names like *photo_count*, *view_count*, *transaction_price*.

This way, the contents of a column are always known from the schema, and are not dependent on other values in the row.

> **select** **sum**(item_value) **as** **photo_count**

> **from** items

> **where** item_type **=** **'Photo Count'**

> *-- vs*

> **select** **sum**(photo_count) **from** items

Don’t prefix column names with the name of the containing table. It’s generally unhelpful to have the users table contain columns like *user_birthday*, *user_created_at*, *user_name*.

Avoid using reserved keywords like *column*, *tag*, and *user* as column names. You’ll have to use extra quotes in your queries and forgetting to do so will get you very confusing error messages. The database can wildly misunderstand the query if a keyword shows up where a column name should be.

### 3. Use Simple, Descriptive Table Names

If the table name is made of up of multiple words, use underscores to separate the words. It’s much easier to read *package_deliveries* than *packagedeliveries*.

And whenever possible, use one word instead of two: *deliveries* is even easier to read.

> **select** ***** **from** packagedeliveries

> *-- vs*

> **select** ***** **from** deliveries

Don’t prefix tables to imply a schema. If you need the table grouped into a scope, put those tables into a schema. Having tables with names like *store_items*, *store_transactions*, *store_coupons*, like prefixed column names, is generally not worth the extra typing.

We recommend using pluralized names for tables (e.g. *packages*), and pluralizing both words in the name of a join table (e.g. *packages_users*). Singular table names are more likely to accidentally collide with reserved keywords and are generally less readable in queries.

### 4. Have an Integer Primary Key

Even if you’re using UUIDs or it doesn’t make sense (e.g. for join tables), add the standard *id* column with an auto-incrementing integer sequence. This kind of key makes certain analyses much easier, like [selecting only the first row of a group](https://www.periscopedata.com/blog/4-ways-to-join-only-the-first-row-in-sql.html).

And if an import job ever duplicates data, this key will be a life-saver because you’ll be able to delete specific rows:

> **delete** **from** my_table

> **where** id **in** (**select** ...) **as** duplicated_ids

Avoid multi-column primary keys. They can be difficult to reason about when trying to write efficient queries, and very difficult to change. Use an integer primary key, a multi-column unique constraint, and several single-column indexes instead.

### 5. Be Consistent with Foreign Keys

There are many styles for naming primary and foreign keys. Our recommendation, and the most popular, is to have a primary key called *id* for any table *foo*, and have all foreign keys be named *foo_id*.

Another popular style uses globally unique key names, where the *foo* table has a primary key called *foo_id* and all foreign keys are also called *foo_id*. This can get confusing or have name collisions if you use abbreviations (e.g. *uid* for the *users table*), so don’t abbreviate.

Whatever style you choose, stick to it. Don’t use *uid* in some places and *user_id* or *users_fk* in others.

> **select** *****

> **from** packages

>   **join** users **on** users.user_id **=** packages.uid

> *-- vs*

> **select** *****

> **from** packages

>   **join** users **on** users.id **=** packages.user_id

> *-- or*

> **select** *****

> **from** packages

>   **join** users **using** (user_id)

And be careful with foreign keys that don’t obviously match up to a table. A column named *owner_id* might be a foreign key to the **users** table, or it might not. Name the column *user_id* or, if necessary, *owner_user_id*.

### 6. Store Datetimes as Datetimes

Don’t store Unix timestamps or strings as dates: convert them to *datetimes* instead. While SQL’s date math functions aren’t the greatest, doing it yourself on timestamps is even harder. Using SQL date functions requires every query to involve a conversion from the timestamp to a *datetime*:

> **select** date(from_unixtime(created_at))

> **from** packages

> *-- vs*

> **select** date(created_at)

> **from** packages

Don’t store the year, month, and day in separate columns. This will make every time series query much harder to write, and will prevent most novice SQL users from being able to use the date information in this table.

> **select** date(created_year **||** **'-'** 

>   **||** created_month **||** **'-'** 

>   **||** created_day)

> *-- vs*

> **select** date(created_at)

### 7. UTC, Always UTC

Using a timezone other than [UTC](http://en.wikipedia.org/wiki/Coordinated_Universal_Time) will cause endless problems. Great tools (including [Periscope](https://www.periscopedata.com/)) have all the functionality you need you convert the data from UTC to your current timezone. In Periscope, it’s as easy as adding *:pst* to convert to from UTC to Pacific Time:

> **select** [created_at:pst], email_address

> **from** users

The database’s time zone should be UTC, and all datetime columns should be types that strip time zones (e.g. **timestamp without time zone**).

If your database’s time zone is not UTC, or you have a mix of UTC and non-UTC datetimes in your database, time series analysis will be a lot harder.

### 8. Have One Source of Truth

There should only ever be one source of truth for a piece of data. Views and rollups should be labeled as such. This way consumers of that data will know there is a difference between the data they are using and the raw truth.

> **select** *****

> **from** daily_usage_rollup

Leaving legacy columns around like *user_id*, *user_id_old*, *user_id_v2* can become an endless source of confusion. Be sure to drop abandoned tables and unused columns during regular maintenance.

### 9. Prefer Tall Tables without JSON Columns

You don’t want to have super-wide tables. If there’s more than a few dozen columns and some of them are named sequentially (e.g. *answer1*, *answer2*, *answer3*), you’re going to have a bad time later.

Pivot the table into a schema that doesn’t have duplicated columns - this schema shape will be a lot easier to query. For example, getting the number of completed answers for a survey:

> **select**

>   **sum**(

> ​    (**case** **when** answer1 **is** **not** **null**

> ​      **then** *1* **else** *0* **end**) **+**

> ​    (**case** **when** answer2 **is** **not** **null**

> ​      **then** *1* **else** *0* **end**) **+**

> ​    (**case** **when** answer3 **is** **not** **null**

> ​      **then** *1* **else** *0* **end**)

>   ) **as** num_answers

> **from** surveys

> **where** id **=** *123*

> *-- vs*

> **select** **count**(response)

> **from** answers

> **where** survey_id **=** *123*

For analysis queries, extracting data from JSON columns can greatly degrade a query’s performance. While there are a lot of great reasons to use JSON columns in production, there aren’t for analysis. Aggressively schematize JSON columns into the simpler data types to make analysis a lot easier and faster.

### 10. Don’t Over-Normalize

Dates, zip codes, and countries don’t need their own tables with foreign key lookups. If you do that, every query ends up with a handful of the same joins. It creates a lot of duplicated SQL and a lot of extra work for the database.

> **select**

>   dates.d,

>   **count**(*1*)

> **from** users

>   **join** dates **on** users.created_date_id **=** dates.id

> **group** **by** *1*

> *-- vs*

> **select**

>   date(created_at),

>   **count**(*1*)

> **from** users

> **group** **by** *1*

Tables are for first class objects that have a lot of their own data. Everything else can be additional columns on a more important object.

1. foreign key columns should have the same name as the primary key to which they refer

   (**update:** obviously now that i just name my primary keys “id”, i name my foreign keys the singular form of the table name + “_id”)

- makes the table to which they refer completely obvious
- if there are multiple foreign keys referencing same table, prefix the foreign key column name with an appropriately descriptive adjective (e.g. lead_person_id, technical_person_id, etc. which transparently reference person_id in the person table)
2. suffix date-type columns with “_on”, suffix datetime-type columns with “_at”, and prefix boolean-type columns with “is_” or “has_”

   (**update:** i now tend to use the “_date” suffix for date-type columns and “_time” for datetime-type columns—even if they are stored as integers)

- prevents confusing with more common text/number data types

- ### Strongly type your data
  
  Just like in programming, there are different types to do different things, use the proper type for the job. In other words, strongly type your data.
  
  - Use a `boolean` or `bit` to hold true/false values
  - Don't use character fields to hold numbers
  - Make sure number fields have constraints for valid values
  - Only use `char/nchar` fields when the data is truly fixed length

**Standard Naming Convention for Tables**
The standard naming convention for tables are as follows:

- It should be in Pascal Case
- It should not have Spaces
- Multiple words should be split with Underscore
- It should be Plural (more than one in number) – Example: Students Table, rather than Student.  If it contains multiple words only last word should be plural.  Example: Students_Photos
- If your database deals with different logical functions and you want to group your tables according to the logical group they belong to, it won’t hurt prefixing your table name with a two or three character prefix that can identify the group. For example, your database has tables which store information about Sales and Human resource departments, you could name all your tables related to Sales department as: SL_Customes, SL_Sales, SL_Orderss
- You could name all your tables related to Human resources department as like: HR_Candidates, HR_PremierInstitutes, HR_InterviewSchedules

**Standard Naming Convention for Fields/Columns**
The standard naming convention for Fields or Columns are as follows:

- It should not have Spaces.
- Multiple words should be split with Underscore.
- It should be Singular – Example: Student_ID column name, rather than Student s_ID or Student_IDS.

**Standard Naming Convention for Views**
The standard naming convention for views are as follows:

- Views not always represent a single entity. A view can be a combination of two or more tables based on a join condition, thus, effectively representing two entities. In this case, consider combining the names of both the base tables. Example: A view combining two tables ‘Students’ and ‘Addresses’, name the view as ”StudentsAddresses”
- Try to avoid using spaces in view name.

**Standard Naming Convention for Stored Procedures or SP**
The standard naming convention for stored procedures are as follows:

- Never prefix your stored procedures with ‘sp_’. If you use it then MS SQL Server first searches the SP in master database, if not found then search current database.
- Procedure name should be defined as TableName_ProcedureFunctionalityName.  Example: Students_SelectAll,  Students_Insert, Students_Update, Students_Delete.  If table name is too long, it is also better to use short name of table rather than full tablename prefix, Example: Emp_SelectAll, Emp_Insert.  If table name contains multiple words like Student_Locations then it is better to give name like SL_SelectAll, SL_Insert.  If short name are getting duplicate, then you can change of one of short name to avoid duplication or confusion.
- If you are creating procedure which is general in nature or combines 2 or more tables or mainly business logic which cannot be associated with any table, then it is better to use as BusinessLogicName_ProcedureFunctionalityName.  Example:  procedure for students quarterly sales report should be named something like Reports_Std_Quaterly_Sales.
- Always try to use such a name which describes the whole functionality of the procedure.

**Standard Naming Convention for Functions**
The standard naming convention for functions are as follows:

- Function name are mostly generic utilities, but incase if they are associated with table, then follow procedure naming convention, else use meaningful name.  Example:  CelsiusToFahrenheit  – If you pass Celsius temperature it will return Fahrenheit temperature.

**Standard Naming Convention for Triggers**
The standard naming convention for triggers are as follows:

- Trigger always depend on base table. So try to use table name with trigger name.
- Triggers are associated with one or more of the following operations: Insert, Update, Delete. So, the name of the trigger should reflect its nature. Example: students_instrg, students_updtrg, students_deltrg.
- If you have a single trigger for more than one action (same trigger for insert and update or update and delete or any such combination), use the words ‘ins’, ‘upd’, ‘del’ together in the name of the trigger. Here’s an example. Example: students_InsUpdtrg.

**Standard Naming Convention for Indexes**
The standard naming convention for indexes are as follows:

- Index name should be name with prefix idx_ColumnName. Example: Idx_Student_ID.

**Standard Naming Convention for Primary Keys**
The standard naming convention for primary keys are as follows:

- Primary key should be name as PK_TableName.  Example:  PK_Students.

**Standard Naming Convention for Foreign Keys**
The standard naming convention for foreign keys are as follows:

- Foreign key should be name as FK_PrimaryTableName_ForeignTableName. Example:  PK_Students_Departments.

![image-20190726160026595](/Users/chenqingze/Library/Application Support/typora-user-images/image-20190726160026595.png)

# Method Naming Conventions

The Date-Time API offers a rich set of methods within a rich set of classes. The method names are made consistent between classes wherever possible. For example, many of the classes offer a `now` method that captures the date or time values of the current moment that are relevant to that class. There are `from` methods that allow conversion from one class to another.

There is also standardization regarding the method name prefixes. Because most of the classes in the Date-Time API are immutable, the API does not include `set` methods. (After its creation, the value of an immutable object cannot be changed. The immutable equivalent of a `set` method is `with`.) The following table lists the commonly used prefixes:

| Prefix   | Method Type    | Use                                                                                                                             |
| -------- | -------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `of`     | static factory | Creates an instance where the factory is primarily validating the input parameters, not converting them.                        |
| `from`   | static factory | Converts the input parameters to an instance of the target class, which may involve losing information from the input.          |
| `parse`  | static factory | Parses the input string to produce an instance of the target class.                                                             |
| `format` | instance       | Uses the specified formatter to format the values in the temporal object to produce a string.                                   |
| `get`    | instance       | Returns a part of the state of the target object.                                                                               |
| `is`     | instance       | Queries the state of the target object.                                                                                         |
| `with`   | instance       | Returns a copy of the target object with one element changed; this is the immutable equivalent to a `set` method on a JavaBean. |
| `plus`   | instance       | Returns a copy of the target object with an amount of time added.                                                               |
| `minus`  | instance       | Returns a copy of the target object with an amount of time subtracted.                                                          |
| `to`     | instance       | Converts this object to another type.                                                                                           |
| `at`     | instance       | Combines this object with another.                                                                                              |