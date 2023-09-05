# Contact Book: Our very first database

In this project, we'll interact with a real database for the first time.

Our goal is to build an application that allows us to store contact information for people we know. The data we want to store will look something like this:

| first_name | last_name | date_of_birth | street_address_1    | street_address_2 | city         | state | zip        | phone              | notes                                   |
|------------|-----------|---------------|---------------------|------------------|--------------|-------|------------|--------------------|-----------------------------------------|
| Carol      | Reynolds  | 20 Oct 2016   | 4556 Mirna Shores   | Apt. 111         | Stromanhaven | DE    | 13654-8312 | 308-571-8066 x3565 | We had a wonderful dinner at La Tavola! |
| Alice      | Boyer     | 25 Oct 1977   | 7774 Gibson Station | Suite 284        | New Seth     | IN    | 70681      | 984.425.6912       | Ask about their mom                     |
| Bob        | Stokes    | 30 Nov 1941   | 21817 Lionel Cliffs |                  | South Hailey | CA    | 32418      | +1-487-497-4041    | Loves tulips                            |
{: .bleed-full }

Etc.

Along the way, we'll learn how to:

- Create a database.
- Create a database table.
- Create a row in the table.
- Query the table in various ways to retrieve rows.
- Update a row.
- Delete a row.

Importantly, we'll learn how to do all of this _in Ruby_, which will allow us to integrate the database into our Ruby apps. Let's get started.

## Setup

Fork this repository and create a codespace. There are no automated tests in this project.

The starting point is [a blank Rails app](https://github.com/appdev-projects/rails-7-template); there's nothing else in it to begin with.

## Databases are command-line tools

If we want our applications to store information permanently on the hard drive of the computer they are running on, and retain the information even if the computer is turned off and back on again, then we have a few options.

We could use the Ruby [File](https://ruby-doc.org/3.2.1/File.html) or [CSV](https://ruby-doc.org/3.2.1/stdlibs/csv/CSV.html) or [PStore](https://ruby-doc.org/3.2.1/stdlibs/pstore/PStore.html) classes to develop our own storage system.

Or, we can leverage a **database** — a piece of software that exists specifically for the purpose of making it easier for apps to store data permanently. Databases are separate from the app itself, so apps written in any language (Ruby, Python, Java, etc) can use any database (PostgreSQL, MySQL, SQLite, etc).

Since databases exist for other _apps_ to use (not for humans to use directly), **the interface to a database is text**. Databases generally don't come with a graphical user interface (GUI) out-of-the-box. Essentially, the apps that we build _are_ the GUI to the database!

<aside>
Some databases come with barebones GUIs, but those GUIs are not intended for end users of an app — they are debugging tools for the developers writing the end user app.
</aside>

Relational databases, the kind of database that powers most apps, use a special language called Structured Query Language (SQL). Any app (whether Ruby, Python, Java, etc) that wants to store data in a relational database must ultimately output SQL commands and send them to the database for processing.

In particular, we're going to use a very powerful and popular open-source database known as PostgreSQL (or "Postgres", for short). We're first going to see how to work with Postgres directly, but we'll soon move on to working with Postgres through Ruby. That will enable us to integrate permanent storage into our Rails apps – finally! Let's get started.

## Working directly with Postgres

Don't worry about memorizing any of the raw Postgres and SQL commands that you'll see in this section. I just want you to have a sense of how the underlying database works before we graduate to using a wonderful Ruby gem called ActiveRecord that makes it all much easier.

First, you have to install the database that you want to use on your computer. Postgres is already installed on our codespaces, so we're set.

Once Postgres is installed, we launch it from a bash prompt with the `psql` command:

```
contact-book main % psql

psql (12.15 (Ubuntu 12.15-0ubuntu0.20.04.1))
Type "help" for help.

postgres=# 
```

If all went well, you should end up at a prompt that looks like `postgres=# `. We can now issue commands to the database at this prompt.

Let's list what databases currently exist in this installation of Postgres on this computer with the `\list` command. You should see something similar to the following:

```
postgres=# \list

                                         List of databases
            Name            |  Owner  | Encoding |   Collate   |    Ctype    |  Access privileges  
----------------------------+---------+----------+-------------+-------------+---------------------
 postgres                   | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 rails_template_development | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 rails_template_test        | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0                  | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/student         +
                            |         |          |             |             | student=CTc/student
 template1                  | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/student         +
                            |         |          |             |             | student=CTc/student
(5 rows)
```

<div class="bg-red-100 py-1 px-5" markdown="1">
**IMPORTANT NOTE:** Sometimes the output of a command-line command is longer than the terminal window can accommodate at once. In that case, you will see a `:▮` at the bottom of the terminal, which means it's waiting for you to scroll through the rest of the output.

You can scroll through one line at a time with <kbd>return</kbd> or one page at a time with <kbd>space</kbd>. Once you reach the end of the output, you will see `(END)`. 

**Then, press <kbd>Q</kbd> to return to whatever prompt you were at before, so that you can issue more commands.**
</div>

We can see that there are already a few databases that exist in the codespace. We'll talk about what these are later.

Let's create our own, brand new database called "my_contact_book" with our first SQL command. Copy-paste the following after the `postgres=#` prompt (usually, I ask you not to copy-paste, and instead to type things out; but for the raw SQL commands in this section, it's okay to copy-paste because we're not trying to build SQL muscle memory right now):

```sql
CREATE DATABASE my_contact_book;
```

`CREATE DATABASE` is the SQL command to start off a new database. The name of the database comes after `DATABASE`; the name should be `snake_case`.

You should see output like this:

```
postgres=# CREATE DATABASE my_contact_book;

CREATE DATABASE
```

And if you try to `\list` the databases again, you should now see our brand new one on the list:

```
postgres=# \list

                                         List of databases
            Name            |  Owner  | Encoding |   Collate   |    Ctype    |  Access privileges  
----------------------------+---------+----------+-------------+-------------+---------------------
 my_contact_book            | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres                   | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 rails_template_development | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 rails_template_test        | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0                  | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/student         +
                            |         |          |             |             | student=CTc/student
 template1                  | student | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/student         +
                            |         |          |             |             | student=CTc/student
(6 rows)
```

We now have a new database called "my_contact_book" ready to go. Let's "enter" our new database so that we can start creating data within it. To "enter" a specific database, we use the `\connect` command at the `postgres=#` prompt:

```
postgres=# \connect my_contact_book

You are now connected to database "my_contact_book" as user "student".

my_contact_book=#
```

You should observe that the prompt has changed from `postgres=#` to `my_contact_book=#`. Now that we're in our new database, we can see what tables there are with the `\dt` command (short for "describe tables"):

```
my_contact_book=# \dt

Did not find any relations.
```

"Relation" is the formal, computer science-y term for what we've been calling a "table" — a set of records. This is where the name "relational database" comes from; _not_ from one-to-many and many-to-many relationships.

We're going to be seeing the term "relation" a lot moving forward — look out for it and get into the habit of thinking **"a 'relation' is 'a _set_ of _multiple records_'"**.
{: .bg-red-100.py-1.px-5 }

It makes sense that our brand new database doesn't contain any relations yet. Let's create a table called "contacts" by issuing some SQL at the `my_contact_book=# ` prompt:

```sql
CREATE TABLE contacts (
  id SERIAL PRIMARY KEY,
  first_name TEXT,
  last_name TEXT,
  date_of_birth DATE,
  street_address_1 TEXT,
  street_address_2 TEXT,
  city TEXT,
  state TEXT,
  zip TEXT,
  phone TEXT,
  notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

If you run that SQL at the `my_contact_book=#` prompt and then `\dt` again, you should now see that we have a relation called `contacts`:

```
my_contact_book=# \dt
          List of relations
 Schema |   Name   | Type  |  Owner  
--------+----------+-------+---------
 public | contacts | table | student
(1 row)
```

Again, **don't worry about memorizing the SQL we're exploring**; we'll be graduating to the Ruby equivalents soon. But let's examine the SQL a little to get a sense of it:

- `CREATE TABLE` is the SQL command to add a new table to our database.
    - The name of the table we want comes after `TABLE`; the name should be `snake_case`.
    - The convention is to make the table name the plural version ("contacts") of what each row represents (one "contact"). But, you could call it anything you want, even "zebra" or "giraffe".
- After the table name are parentheses which contains a list of all the columns we want to add to the table.
    - Column names should be `snake_case`.
    - Each column name is followed by the type of data we plan to store in that column. The most common datatypes are:
        - `TEXT`: Similar to Ruby `String`.
        - `INTEGER`: Similar to Ruby `Integer`.
        - `BOOLEAN`: Can hold one of the three possible values — `true`, `false`, or `nil`.
        - `DATE`: For values with a date part but no time part.
        - `TIME`: For values with a time part but no date part.
        - `TIMESTAMP`: For values that have both date and time parts.
        - `DECIMAL`: Similar to Ruby `Float`.
    - `id SERIAL PRIMARY KEY` is how we specify what we want to call the primary key column (the convention is to always call the primary key `id`).
        - The `SERIAL` part indicates that we want the database to automatically assign this value to each row for us, starting with `1` and going up from there.
    - `created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP` creates a column called `created_at` of type `TIMESTAMP` and asks the database to automatically assign the current time when a row is inserted into the table.

We've created a database, and added a table to the database. Let's see what's in the table with the `SELECT * FROM contacts;` command:

```
my_contact_book=# SELECT * FROM contacts;

 id | first_name | last_name | date_of_birth | street_address_1 | street_address_2 | city | state | zip | phone | notes | created_at 
----+------------+-----------+---------------+------------------+------------------+------+-------+-----+-------+-------+------------
(0 rows)
```

We can see that there are currently 0 rows in our contacts table. Makes sense, since we haven't added any yet!

`SELECT` is the command we use for most of our READ operations. The `*` indicates that we want all of the columns; we could also be more specific and select e.g. only the `first_name` and `last_name` columns:

```
my_contact_book=# SELECT first_name, last_name FROM contacts;

 first_name | last_name 
------------+-----------
(0 rows)
```

But usually we'll `SELECT * FROM <table name>` to get all of the columns.

Okay, let's finally add some records to our table! We'll use the `INSERT` command for that. Try running the following SQL:

```sql
INSERT INTO contacts (
  first_name, 
  last_name, 
  date_of_birth, 
  street_address_1, 
  street_address_2, 
  city, 
  state, 
  zip,
  phone,
  notes
) VALUES (
  'Carol', 
  'Reynolds', 
  '2016-10-20', 
  '4556 Mirna Shores', 
  'Apt. 111', 
  'Stromanhaven', 
  'DE', 
  '13654-8312',
  '308-571-8066 x3565',
  'We had a wonderful dinner at La Tavola!'
);
```

If you copy-paste the above into `psql` and then `SELECT * FROM contacts;` again, you should finally see some data in our table:

```
 id | first_name | last_name | date_of_birth | street_address_1  | street_address_2 |     city     | state |    zip     |       phone        |                  notes                  |          created_at           
----+------------+-----------+---------------+-------------------+------------------+--------------+-------+------------+--------------------+-----------------------------------------+-------------------------------
  1 | Carol      | Reynolds  | 2016-10-20    | 4556 Mirna Shores | Apt. 111         | Stromanhaven | DE    | 13654-8312 | 308-571-8066 x3565 | We had a wonderful dinner at La Tavola! | 2023-07-27 20:04:31.365298+00
(1 row)
```

**Remember to press <kbd>Q</kbd> to get back to your prompt if the output of a command is too long for the terminal window.**

Notice that an `id` and `created_at` were automatically populated. You could shut down your codespace, come back tomorrow, and this data would still be there. Finally, persistent storage!

## Interacting with Postgres from within a Rails app

Everything we saw in the previous section had nothing to do with Ruby on Rails; we were working directly with Postgres through its built-in `psql` program.

Now let's see how we can get our Rails app talking to Postgres, so that we can use Postgres to permanently store information that our users send us in HTTP requests.

### config/database.yml

All Rails apps come out-of-the-box with a file called `config/database.yml`. `.yml` is the extension for a markup language called "Yet Another Markup Language". It's supposed to be easy to type, sort of like Markdown; but highly structured, sort of like JSON. `.yml` files are often used for configuration and settings.

Among other things, `config/database.yml` is how we tell Rails which database we want it to connect to. Currently, on Line 26, the database is set to `rails_template_development` — this is a default name that was automatically generated when I first created this blank Rails app for us.

Let's ask Rails to connect to the database that we created instead. Edit Line 26 of `config/database.yml` to be:

```yml
database: my_contact_book
```

In your terminal, quit out of `psql` with the `quit` or `\q` command, or open a new terminal tab; and at a bash prompt, run the command `rails dbconsole`. You should see output like this:

```
contact-book main % rails dbconsole
psql (12.15 (Ubuntu 12.15-0ubuntu0.20.04.1))
Type "help" for help.

my_contact_book=#
```

What just happened? The `rails dbconsole` command does two things for us:

- It launches `psql`.
- It automatically `\connect`s to the table specified in `config/database.yml`.

Just like before, we can see what tables are in the database with `\dt`:

```
my_contact_book=# \dt

          List of relations
 Schema |   Name   | Type  |  Owner  
--------+----------+-------+---------
 public | contacts | table | student
(1 row)
```

And we can see the table and data that we previously created. **So: connecting a database to Rails is a matter of editing the `config/database.yml` file.**

### The /rails/db GUI

Now that we've connected our database to Rails, we can take advantage of a handy gem: Rails DB. Rails DB provides a GUI to our database, to make it easier for developers (not end users) to see what data is in the database:

- Start your web server with `bin/dev`.
- Open the live app preview. You should see the default Rails homepage, since we haven't defined a root route yet.
- Manually navigate to the URL `/rails/db`. This route is already built for us by the Rails DB gem.

You should see a visual interface to the database:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690491792/rails-db-1_s2v9iu.png)

In the left sidebar is a list of the tables in the database. Click "contacts" to see the rows currently in that table:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690491846/rails-db-2_uanp7x.png)

If you click the blue "+ ADD" button on the top-right, Rails DB will provide a basic form that you can use to add more rows:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690491901/rails-db-3_ebviou.png)

You can also use the SQL Editor to issue raw SQL commands, or Export the data as a CSV. Pretty neat!

We'll use the Rails DB GUI often to get a quick visual view into our database tables. But, what we really need to do is learn how to CRUD data into our tables using Ruby. Then and only then will we be able to use the database from within our actions, which is our end goal!

## ActiveRecord

In the next project, we're going to start building the app whose database we've already designed, Must See Movies.

In that project, the four tables we planned — movies, directors, characters, and actors — have already been created. I did that just to avoid e.g. you making any typos in column names while creating the tables, which would make it hard for you to follow along.

Given an already-created database and tables, we'll learn how we can use the wonderful gem ActiveRecord to interact with those tables in Ruby. ActiveRecord translates our Ruby into highly performant, secure SQL automatically.
