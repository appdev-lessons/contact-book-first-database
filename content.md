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

Importantly, we'll learn how to do this directly within the database and also _in Ruby_, which will allow us to integrate the database into our Ruby apps. Let's get started.

## Setup

<div class="bg-blue-100 py-1 px-5" markdown="1">

The GitHub repository associated with this lesson is just for you to experiment with a database. There are also quiz questions throughout this lesson. You will need to answer those questions to check your understanding and get a grade.
</div>

[Fork this repository](https://github.com/appdev-projects/contact-book/fork) and create a codespace. There are no automated tests in this project.

The starting point is a blank Rails app; there's nothing else in it to begin with.

## Databases are command-line tools

If we want our applications to store information permanently on the hard drive of the computer they are running on, and retain the information even if the computer is turned off and back on again, then we have a few options.

We could use the Ruby [File](https://ruby-doc.org/3.2.1/File.html) or [CSV](https://ruby-doc.org/3.2.1/stdlibs/csv/CSV.html) or [PStore](https://ruby-doc.org/3.2.1/stdlibs/pstore/PStore.html) classes to develop our own system that persists data to the hard drive. But, writing a secure, scalable, and performant storage system would be a massive undertaking unto itself, apart from writing our actual app.

Instead, we can outsource that job to a **database** — a piece of software that exists specifically for the purpose of making it easier for apps to store data permanently. Databases are separate from the app itself, so apps written in any language (Ruby, Python, Java, etc) can use any database (PostgreSQL, MySQL, SQLite, etc).

Since databases exist for other _apps_ to use (not for humans to use directly), **the interface to a database is text**. Databases generally don't come with a graphical user interface (GUI) out-of-the-box. (Essentially, a web app that we build _is_ the graphical user interface to the underlying database!)

<aside>
Some databases come with barebones GUIs, but those GUIs are not intended for end users of an app — they are debugging tools for the developers writing the end user app.
</aside>

Relational databases, the kind of database that powers most apps, use a special language called Structured Query Language (SQL). Any app (whether Ruby, Python, Java, etc) that wants to store data in and retrieve data from a relational database must ultimately output SQL commands and send them to the database for processing.

In particular, we're going to use a very powerful and popular open-source database known as PostgreSQL (or "Postgres", for short). We're first going to see how to work with Postgres directly, but we'll soon move on to working with Postgres through Ruby. That will enable us to integrate permanent storage into our Rails apps – finally! Let's get started.

- Select all that are true:
- Structured Query Language (SQL) is only used for Ruby apps.
  - Not quite, re-read the previous section.
- SQL is the language of databases.
  - Yes!
- Databases are part of a Rails app and can't be used by other types of apps.
  - Not quite, re-read the previous section.
- Databases are a piece of software separate from our app.
  - Yes!
- Our apps provide an interface to interact with a database.
  - Yes!
{: .choose_all #databases_and_sql title="Databases and SQL" points="3" answer="[2,4,5]" }

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

Let's create our own, brand new database called "my_contact_book" with our first SQL command. Copy-paste the following after the `postgres=#` prompt (usually, I ask you not to copy-paste, and instead to type things out; but for the raw SQL commands in this section, it's okay to copy-paste because we're not trying to build SQL muscle memory just yet):

```sql
CREATE DATABASE my_contact_book;
```

`CREATE DATABASE` is the SQL command to start off a new database. The name of the database comes after `DATABASE`; the name should be `snake_case`.

Don't forget the semicolon at the end of the SQL statement; it is required and the command won't work without it. If all went well, you should see the output `CREATE DATABASE`:

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

"Relation" is the formal, computer science-y term for what we've been calling a "table" — a set of records (formally known as "tuples"). The concept of a "relation" is where the name "relational database" comes from. The name does _not_ come from the concept of one-to-many and many-to-many associations between records, confusingly also known as "relationships".

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

Again, **don't worry about memorizing the SQL we're exploring**; we're not trying to learn SQL syntax right now, and we'll be graduating to the Ruby equivalents soon. But let's examine the SQL a little to get a sense of it:

- `CREATE TABLE` is the SQL command to add a new table to our database.
    - The name of the table we want comes after `TABLE`; the name should be `snake_case`.
    - The convention is to make the table name the plural version ("contacts") of what each row represents (one "contact"). But, you could call the table anything you want.
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

Okay, let's finally add a record to our table! We'll use the `INSERT` command for that. Try running the following SQL; it's pretty cumbersome, so it's okay to copy-paste it:

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

Notice that values for `id` and `created_at` were automatically assigned. You could shut down your codespace, come back tomorrow, and this data would still be there. Finally, persistent storage!

- Select all that are true:
- A "relation" is a single record from the database.
  - Not quite, re-read the previous section.
- A "relation" is a set of records from the database.
  - Yes!
- We usually think of a "relation" as a "table" in our database.
  - Yes!
- Postgres is a language.
  - Not quite.
- Postgres is a database software that we interact with using the SQL language.
  - Yes!
- Persistent storage allows us to permanently store database records.
  - Yes!
{: .choose_all #relations_and_postgres title="Relations and Postgres" points="4" answer="[2,3,5,6]" }

## Interacting with Postgres from within a Rails app

Everything we saw in the previous section had nothing to do with Ruby or Rails; we were working directly with Postgres through its built-in `psql` program.

Now let's see how we can get our Rails app talking to Postgres, so that we can use Postgres to permanently store information that our users send us in HTTP requests.

### config/database.yml

All Rails apps come out-of-the-box with a file called `config/database.yml`. `.yml` is the extension for a markup language called "Yet Another Markup Language". It's supposed to be easy to type, sort of like Markdown; but highly structured, sort of like JSON. `.yml` files are often used for configuration and settings.

<aside markdown="1">
When we deploy our apps with Render, we use a file named `render.yaml` for configuration. The extension `.yml` and `.yaml` are equivalent file endings: both indicate the file is of "Yet Another Markup Language" (YAML) type.
</aside>

Among other things, `config/database.yml` is how we tell Rails which database we want it to connect to. Currently, on Line 27, the database is set to `rails_template_development` — this is a default name that was automatically generated when I first created this blank Rails app for us.

Let's ask Rails to connect to the database that we created instead. Edit Line 27 of `config/database.yml` to be:

```yml{6:(13-24)}
# ...

development:
  adapter: sqlite3
  <<: *default
  database: contact_book

# ...
```

In your terminal, quit out of `psql` with the `\q` command, or open a new terminal tab; and at a bash prompt, run the command `rails dbconsole`. You should see output like this:

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

And we can see the table and data that we previously created. **So: we update the `config/database.yml` file to connect Rails to a specific database.**

- The `config/database.yml` file in a Rails app
- allows us to connect to any Postgres database.
  - Yes!
- should never be edited.
  - Not quite. You should have edited it in the previous step
{: .choose_best #config_database title="config/database.yml" points="1" answer="1" }

### The /rails/db GUI

Now that we've connected our database to Rails, we can take advantage of a handy gem: Rails DB. Rails DB provides a GUI to our database, to make it easier for developers (not end users) to see what data is in the database:

- Start your web server with `bin/dev`.
- Open the live app preview. You should see the default Rails homepage, since we haven't defined a root route yet.
- Manually navigate to the URL `/rails/db`. The route, controller, action, and view for this URL is provided for us by the Rails DB gem.

You should see a page that looks like this:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690491792/rails-db-1_s2v9iu.png)

In the left sidebar, there's a list of all the tables that are currently in the database. Click "contacts" to see the rows that are currently in that table:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690491846/rails-db-2_uanp7x.png)

If you click the blue "+ ADD" button on the top-right, Rails DB will provide a basic form that you can use to add more rows:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690491901/rails-db-3_ebviou.png)

You can also use the SQL Editor to issue raw SQL commands, or Export the data as a CSV. Pretty neat!

We'll use the Rails DB GUI often to get a quick visual view into our database tables. But, what we really need to do is learn how to CRUD data into our tables using Ruby. Then and only then will we be able to write code to CRUD data from within our actions, which is our end goal!

- Did you check out the `/rails/db` page in your live app preview?
- Yes, it's really useful.
  - Good! Thanks, Ruby community for this awesome gem!
- No, I'm just reading.
  - Do or do not, there is no read!
{: .choose_best #rails_db title="rails/db" points="1" answer="1" }

## ActiveRecord

Fortunately for us, there's a wonderful Ruby gem called ActiveRecord (written by the authors of Ruby on Rails) that automates the heavy lifting of connecting to a database and performing CRUD operations. Let's see how we can use ActiveRecord to interact with the `contacts` table that we created.

### Models

For each table that we want to interact with, we will create a Ruby class to act as a translator to that table. This Ruby class will generate SQL, send it to the database, and transform the data sent back into Ruby objects that are easy for us to work with. We refer to these classes  as "models", and we place them in the `app/models` folder.

Let's create a class called `Contact` to deal with the `contacts` table for us.

At this point, you should stop copy-pasting and instead start typing out the examples again. We very much want to build muscle memory around using ActiveRecord.
{: .bg-red-100.py-1.px-5 }

Create a new file called `contact.rb` in the `app/models/` folder in your codespace, and fill it in with the following:

```ruby
# app/models/contact.rb

class Contact
end
```

If we:

1. Put a Ruby class in `app/models`.
2. Name the file the same thing as the class (but the `snake_case`d version of the name, since it's a file).

Then Rails will automatically `require` that class in _every_ other file in our app. Phew! That will save us from typing hundreds of `require` statements.

Notice that we named the class singularly (`Contact`), rather than plurally (`Contacts`). This is because each instance of this class is going to represent one **row** from the contacts table. It's similar to how the `String` class is called `String`, not `Strings`.

- Select all that are true:
- The `app/models/` folder contains our database software.
  - Not quite, re-read the previous section.
- The files in the `app/models/` folder allow us to interact with our database relations (a.k.a. "tables").
  - Yes!
- Our database table is called `Contacts` and our model is also `Contacts`.
  - Not quite, re-read the previous section.
- Our database table is called `contacts` and our model is called `Contacts`.
  - Not quite, re-read the previous section.
- Our database table is called `contacts` and our model is called `Contact`.
  - Yes!
{: .choose_all #first_model title="First model" points="2" answer="[2,5]" }

### rails console

Let's start to experiment with the `Contact` class. Open an IRB session at a bash prompt with `irb`. Within IRB, try creating a new instance of the `Contact` class with `Contact.new`. You should see an error message:

```
contact-book main % irb

3.2.1 :001 > Contact.new
(irb):1:in `<main>': uninitialized constant Contact (NameError)
        from /home/student/.rvm/ruby-3.2.1/gems/irb-1.6.4/exe/irb:9:in `<top (required)>'
        from /home/student/.rvm/ruby-3.2.1/bin/irb:25:in `load'
        from /home/student/.rvm/ruby-3.2.1/bin/irb:25:in `<main>'
```

IRB can't find the `Contact` class, because we didn't `require` the file yet. Try it again after `require "./app/models/contact"`:

```
3.2.1 :002 > require "./app/models/contact"
 => true
3.2.1 :003 > Contact.new
 => #<Contact:0x00007f389975b448>
```

Now that we've `require`d it, we are able to use the `Contact` class in this IRB session.

Rails includes a very handy tool that saves us the trouble of `require`ing all of our models in IRB sessions: the Rails Console.

`exit` from IRB or open a new terminal tab, and at a bash prompt run the command `rails console` (or `rails c`, for short):

```
contact-book main % rails console
Loading development environment (Rails 7.0.4.3)
[1] pry(main)>
```

Notice the new prompt: `[1] pry(main)> `, and also notice that it said "Loading development environment (Rails 7.0.4.3)".

The Rails Console is very similar to IRB, but _it automatically `require`s all of the files and gems in the Rails app_. This includes gems like `activesupport`, `json`, etc, in addition to all of our own classes from the `app/models/` folder. This is a huge time saver when we're experimenting!

So we can right away start using `Contact` in `rails console` without having to `require` it:

```
contact-book main % rails console
Loading development environment (Rails 7.0.4.3)
[1] pry(main)> Contact.new
=> #<Contact:0x00007f5174607ee0>
```

From now on, when we want a quick way of executing one line of Ruby at a time, we'll use `rails c`  rather than `irb`.

### Inheriting from `ActiveRecord::Base`

As of now, the `Contact` class isn't very interesting. But let's give it superpowers by inheriting from a class called `ActiveRecord::Base`:

```ruby
# app/models/contact.rb

class Contact < ActiveRecord::Base
end
```

Voilà! The `Contact` class has now inherited _hundreds_ of methods that will help us CRUD data in the `contacts` table.

#### Reloading `rails console`

The first inherited method we'll try in `rails console` is `Contact.count`:

```
[2] pry(main)> Contact.count
NoMethodError: undefined method `count' for Contact:Class
from (pry):2:in `__pry__'
```

Uh oh! Ruby complains that there is no method called `Contact.count`. But there is, I promise!

The issue is that **`rails console` doesn't know about edits that we make to model files until we restart it**. In this case, `rails console` doesn't know that we updated the `Contact` class to inherit from `ActiveRecord::Base`.

So `exit`, start a fresh `rails c` session, and then try `Contact.count` again:

```
[3] pry(main)> exit
contact-book main % rails c
Loading development environment (Rails 7.0.4.3)
[1] pry(main)> Contact.count
  Contact Count (14.6ms)  SELECT COUNT(*) FROM "contacts"
=> 1
```

And now it works! We can see from the output that the `.count` method issues some SQL to the database, transforms the result into a Ruby object (an `Integer`, in this case), and then returns it.

### Configuring the table name

You might be wondering, "How did the `Contact.count` method know to use the table name `contacts` when it was writing that SQL statement?"

ActiveRecord automatically _infers_ the table name that we want to interact with based on the name we picked for the model class. Try creating a second model called `Zebra`:

```ruby
# app/models/zebra.rb

class Zebra < ActiveRecord::Base
end
```

If you `exit`, launch a fresh `rails c` to pick up the new file,
and then try `Zebra.count` , you'll see an error:

```
contact-book main % rails c
Loading development environment (Rails 7.0.4.3)
[1] pry(main)> Zebra.count
ActiveRecord::StatementInvalid: PG::UndefinedTable: ERROR:  relation "zebras" does not exist
```

The error "relation 'zebras' does not exist" makes sense, since we didn't create a table by that name in our database. But you can see that ActiveRecord tries to guess the table name based on the name of the class.

If we wanted to, we could specify the table name with the `self.table_name` method:

```ruby
# app/models/zebra.rb

class Zebra < ActiveRecord::Base
  self.table_name = "contacts"
end
```

Now if you `exit`, `rails c`, and try again, it should work:

```
[1] pry(main)> Zebra.count
  Zebra Count (0.4ms)  SELECT COUNT(*) FROM "contacts"
=> 1
```

We now have two models, `Contact` and `Zebra`, which both interact with the same underlying table — `contacts`. In a real app we would never do that, but I'm just proving a point! From now on, we'll always name our model after the table we want it to represent so that we can skip the `self.table_name` step.

This is an example of Rails' philosophy of "convention over configuration". If you follow conventional patterns (like naming your model the same thing as your table), then Rails will by default _just work_ without us having to specify every little thing. But if you want to break from convention, Rails always gives you a way to do that too.

---

**Checkpoint:**

So far I've been doing a lot of explaining, and we've been doing a lot of experimenting in `psql` and `rails console`; but we've actually written very little code. Functionally, here's what we've done so far:

First, I launched `psql` from a bash prompt.

In `psql`, I created a database with the command:

```sql
CREATE DATABASE my_contact_book;
```

Then, I conncected to that database with the command:

```
\connect my_contact_book
```

Then, I created a table called "contacts" with the command:

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

Then, we configured our Rails application to use this database by editing Line 26 of `config/database.yml` to:

```yml
database: my_contact_book
```

Then, we created a model called `Contact:

```ruby
# app/models/contact.rb

class Contact < ActiveRecord::Base
end
```

(We also created a second model called `Zebra`, but that was just for experimentation.)

And that's it!

- [Here you can see the changes that I've made so far.](https://github.com/raghubetina/contact-book/commit/5f1cd87babdbde964d822c4007eff8355b6de1e2)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/contact-book/tree/5f1cd87babdbde964d822c4007eff8355b6de1e2)

### Creating a new record

Let's add another row to the `contacts` table. ActiveRecord makes it straightforward:

- First, we create a new instance of the `Contact` class with `Contact.new`.
- We assign that instance to a variable so that we can continue to work with it — for example, a variable named `x`.
- For each column in the underlying table, **ActiveRecord automatically adds attribute accessor methods for us**. So we can right away start assigning values to each column like `x.first_name = "Alice"`.
- Once I'm done assigning attribute values, I call the `.save` method. This method will issue the SQL to actually insert the row into the underlying table.

Here's how it looks in action:

```
[1] pry(main)> x = Contact.new
=> #<Contact:0x00007efe0b7e0030 id: nil, first_name: nil, last_name: nil, date_of_birth: nil, street_address_1: nil, street_address_2: nil, city: nil, state: nil, zip: nil, phone: nil, notes: nil, created_at: nil>

[2] pry(main)> x.first_name = "Alice"
=> "Alice"

[3] pry(main)> x.last_name = "Boyer"
=> "Boyer"

[4] pry(main)> x.save
  TRANSACTION (0.6ms)  BEGIN
  Contact Create (13.8ms)  INSERT INTO "contacts" ("first_name", "last_name", "date_of_birth", "street_address_1", "street_address_2", "city", "state", "zip", "phone", "notes", "created_at") VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11) RETURNING "id"  [["first_name", "Alice"], ["last_name", "Boyer"], ["date_of_birth", nil], ["street_address_1", nil], ["street_address_2", nil], ["city", nil], ["state", nil], ["zip", nil], ["phone", nil], ["notes", nil], ["created_at", "2023-07-28 19:17:17.932364"]]
  TRANSACTION (6.4ms)  COMMIT
=> true
```

Notice that the `x.save` method, when called, writes a SQL `INSERT INTO` statement similar to the one we ran directly within `psql`. That means we won't need to type out those cumbersome statements manually ever again!

If you examine `x` now:

```
[5] pry(main)> x
=> #<Contact:0x00007efe0b7e0030
 id: 2,
 first_name: "Alice",
 last_name: "Boyer",
 date_of_birth: nil,
 street_address_1: nil,
 street_address_2: nil,
 city: nil,
 state: nil,
 zip: nil,
 phone: nil,
 notes: nil,
 created_at: Fri, 28 Jul 2023 19:17:17.932364300 UTC +00:00>
```

Observe that the `id` and `created_at` columns have been automatically assigned. The record has been saved to the table! Verify using `Contact.count`, as well as the `/rails/db` GUI:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690572226/rails-db-4_oxhoru.png)

We've inserted data into our database using Ruby!

Let's add another record:

```
[6] pry(main)> y = Contact.new
=> #<Contact:0x00007f172169e4e0 id: nil, first_name: nil, last_name: nil, date_of_birth: nil, street_address_1: nil, street_address_2: nil, city: nil, state: nil, zip: nil, phone: nil, notes: nil, created_at: nil>

[7] pry(main)> y.first_name = "Bob"
=> "Bob"

[8] pry(main)> y.last_name = "Stokes"
=> "Stokes"

[9] pry(main)> y.save
  TRANSACTION (0.3ms)  BEGIN
  Contact Create (25.3ms)  INSERT INTO "contacts" ("first_name", "last_name", "date_of_birth", "street_address_1", "street_address_2", "city", "state", "zip", "phone", "notes", "created_at") VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11) RETURNING "id"  [["first_name", "Bob"], ["last_name", "Stokes"], ["date_of_birth", nil], ["street_address_1", nil], ["street_address_2", nil], ["city", nil], ["state", nil], ["zip", nil], ["phone", nil], ["notes", nil], ["created_at", "2023-07-28 20:52:04.477159"]]
  TRANSACTION (7.3ms)  COMMIT
=> true

[10] pry(main)> Contact.count
  Contact Count (0.8ms)  SELECT COUNT(*) FROM "contacts"
=> 3
```

### Creating sample data

Over the new few sections, we're going to learn how to find and retrieve records from our `contacts` table. Right now, in my `contacts` table, I only have 3 rows. You might have more or less. But it would be nice to have hundreds of rows in the table before we practice things like sorting, searching, etc.

It would be very tedious to create hundreds of records by typing them one by one into `rails console` like we've been doing. I'm way too lazy for that! But now that we know how to insert records using Ruby, we can automate the process by writing a program.

Let's write a Ruby script that will create a few hundred rows in the contacts table and populate them with random data. We call data like this "sample data", and it's extremely helpful to have while developing an app.

#### Custom rake tasks

Rails provides a place to put Ruby scripts: the `lib/tasks` folders. Let's create a file in that folder called `i_am_lazy.rake`. Notice that the file extension is `.rake` rather than `.rb`. The contents of the file will still be a Ruby program, but these special scripts are known as "rake tasks".

Within the file, type the following code:

```ruby
# lib/tasks/i_am_lazy.rake

task(:howdy) do
  pp "Hello!"
end
```

Then, at a bash prompt, run the command `rake howdy`:

```
contact-book main % rake howdy
"Hello!"
```

What just happened?

- Within `.rake` files, we can use a special method: `task`.
- The argument to `task` should be a `Symbol` — this is what we want to name the task.
- We write however much code we want to for the task within the `do`/`end` of a block.
- We can easily run the task from the bash prompt with the `rake <task name>` command. Rails finds the task by that name, even though it's deeply buried within a file somewhere within the `lib/` folder.

We can add multiple tasks to the same file:

```ruby
# lib/tasks/i_am_lazy.rake

task(:howdy) do
  pp "Hello!"
end

task(:world) do
  pp "World!"
end
```

And the `rake` command will find it:

```
contact-book main % rake howdy
"Hello!"
contact-book main % rake world
"World!"
```

If you try to run a rake task that doesn't exist, you'll get a descriptive error message:

```
contact-book main % rake zebra
rake aborted!
Don't know how to build task 'zebra' (See the list of available tasks with `rake --tasks`)
```

Okay, so: if we want to write standalone Ruby programs within a Rails app, rather than creating a `.rb` file and running it with the `ruby` command, a convenient place to put them is `lib/tasks` and a convenient way to run them is with the `rake` command.

Let's write a task called "sample_contacts" that will make it easier to populate our contacts table.

We'll begin by writing some Ruby that will automate creating just one record, using the same Ruby that we proved works in the `rails console`:

```ruby
# lib/tasks/i_am_lazy.rake

task(:sample_contacts) do
  x = Contact.new
  x.first_name = "Minnie"
  x.last_name = "Mouse"
  x.date_of_birth = "November 18, 1928"
  x.save
end
```

If we run this task right now with `rake sample_contacts`, we'll see an error because we didn't `require` the `Contact` class:

```
contact-book main % rake sample_contacts
rake aborted!
NameError: uninitialized constant Contact
```

Happily, there's a built in way to automatically `require` all of our models, all of the gems in our `Gemfile`, etc. We just have to add `=> :environment` after the task's name:

```ruby
# lib/tasks/i_am_lazy.rake

task(:sample_contacts => :environment) do
  x = Contact.new
  x.first_name = "Minnie"
  x.last_name = "Mouse"
  x.date_of_birth = "November 18, 1928"
  x.save
end
```

The task should now work. Each time we run `rake sample_contacts`, a new record for Minnie Mouse will be inserted.

Let's add Mickey Mouse to the task:

```ruby
# lib/tasks/i_am_lazy.rake

task(:sample_contacts => :environment) do
  x = Contact.new
  x.first_name = "Minnie"
  x.last_name = "Mouse"
  x.date_of_birth = "November 18, 1928"
  x.save

  x = Contact.new
  x.first_name = "Mickey"
  x.last_name = "Mouse"
  x.date_of_birth = "May 15, 1928"
  x.save
end
```

Since I've already saved the contact for Minnie to the database, it's okay to re-use the variable `x` for another contact. I can always look up the contact for Minnie in the database later if I need it.

#### The Faker gem

We can add a loop to our code to create a bunch of records all at once:

```ruby
# lib/tasks/i_am_lazy.rake

task(:sample_contacts => :environment) do
  200.times do
    x = Contact.new
    x.first_name = "Bob"
    x.last_name = "Stokes"
    x.save
  end

  x = Contact.new
  x.first_name = "Minnie"
  x.last_name = "Mouse"
  x.date_of_birth = "November 18, 1928"
  x.save

  x = Contact.new
  x.first_name = "Mickey"
  x.last_name = "Mouse"
  x.date_of_birth = "May 15, 1928"
  x.save
end
```

Now if we run the task, we get a whole bunch of "Bob Stokes" in our table:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690648289/rails-db-6_hodd0e.png)

This is better than having just two or three contacts, but it would be even better if they weren't all duplicates. How can we quickly create a bunch of realistic, varied records?

If we happen to have real data, perhaps in a CSV, parsing it would be a great way to create realistic sample data. Or maybe there's an API we could pull from. Or, perhaps there's a website we could scrape for realistic data.

Another approach is generating random values. Generating random _numbers_ is easy with methods like `rand`, but how could we generate other random values, like names?

There's a gem for that: the Faker gem. I use the Faker gem in almost every project to generate realistic sample data, so I've already included it in our `Gemfile`. We can start using it right away — pop open a `rails console` and try out [the `Faker::Name` class](https://github.com/faker-ruby/faker/blob/main/doc/default/name.md):

```
contact-book main % rails c
Loading development environment (Rails 7.0.4.3)
[1] pry(main)> Faker::Name.first_name
=> "Neal"
[2] pry(main)> Faker::Name.first_name
=> "Joseph"
[3] pry(main)> Faker::Name.first_name
=> "Dorine"
[4] pry(main)> Faker::Name.first_name
=> "Cristin"
[5] pry(main)> Faker::Name.first_name
=> "Alethea"
[6] pry(main)> Faker::Name.first_name
=> "Lisbeth"
```

We see that the `Faker::Name.first_name` is sort of like `rand`, but it generates random names rather than numbers. Let's upgrade our `sample_contacts` task with `Faker::Name.first_name` and `Faker::Name.last_name`:

```ruby
# lib/tasks/i_am_lazy.rake

task(:sample_contacts => :environment) do
  200.times do
    x = Contact.new
    x.first_name = Faker::Name.first_name
    x.last_name = Faker::Name.last_name
    x.save
  end

  x = Contact.new
  x.first_name = "Minnie"
  x.last_name = "Mouse"
  x.date_of_birth = "November 18, 1928"
  x.save

  x = Contact.new
  x.first_name = "Mickey"
  x.last_name = "Mouse"
  x.date_of_birth = "May 15, 1928"
  x.save
end
```

Each time we run our task now, 200 contacts with random names will be added:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690649276/rails-db-7_ustjom.png)

The Faker gem [includes _a lot_ of methods for generating various kinds of values](https://github.com/faker-ruby/faker#generators). Let's flesh out our task by assigning randomized values for all the other columns too, using methods from [the `Faker::Address` class](https://github.com/faker-ruby/faker/blob/main/doc/default/address.md), [the `Faker::Date` class](https://github.com/faker-ruby/faker/blob/main/doc/default/date.md), [the Faker::PhoneNumber class](https://github.com/faker-ruby/faker/blob/main/doc/default/phone_number.md), and [the Faker::Movies::HarryPotter` class](https://github.com/faker-ruby/faker/blob/main/doc/movies/harry_potter.md):

```ruby
# lib/tasks/i_am_lazy.rake

task(:sample_contacts => :environment) do
  200.times do
    x = Contact.new

    x.first_name = Faker::Name.first_name
    x.last_name = Faker::Name.last_name
    x.date_of_birth = Faker::Date.birthday(min_age: 0, max_age: 120)
    x.street_address_1 = Faker::Address.street_address
    x.street_address_2 = Faker::Address.secondary_address
    x.city = Faker::Address.city
    x.state = Faker::Address.state_abbr
    x.zip = Faker::Address.zip
    x.phone = Faker::PhoneNumber.phone_number
    x.notes = Faker::Movies::HarryPotter.quote

    x.save
  end

  x = Contact.new
  x.first_name = "Minnie"
  x.last_name = "Mouse"
  x.date_of_birth = "November 18, 1928"
  x.save

  x = Contact.new
  x.first_name = "Mickey"
  x.last_name = "Mouse"
  x.date_of_birth = "May 15, 1928"
  x.save
end
```

(I couldn't find a perfect method in Faker for generating random, realistic notes for each contact, so I went with a Harry Potter quote instead. Feel free to experiment with something different!)

Now if we run the task, we get fully fleshed out sample contacts in our table:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690650217/rails-db-8_bwjnvu.png)

One last detail: it might be nice to "reset" the table by deleting all of the records before creating new, randomized records. In order to do that, we can use the somewhat dangerous `destroy_all` method:

```ruby
# lib/tasks/i_am_lazy.rake

task(:sample_contacts => :environment) do
  if Rails.env.development?
    Contact.destroy_all
  end

  200.times do
    x = Contact.new

    x.first_name = Faker::Name.first_name
    x.last_name = Faker::Name.last_name
    x.date_of_birth = Faker::Date.birthday(min_age: 0, max_age: 120)
    x.street_address_1 = Faker::Address.street_address
    x.street_address_2 = Faker::Address.secondary_address
    x.city = Faker::Address.city
    x.state = Faker::Address.state_abbr
    x.zip = Faker::Address.zip
    x.phone = Faker::PhoneNumber.phone_number
    x.notes = Faker::Movies::HarryPotter.quote

    x.save
  end

  x = Contact.new
  x.first_name = "Minnie"
  x.last_name = "Mouse"
  x.date_of_birth = "November 18, 1928"
  x.save

  x = Contact.new
  x.first_name = "Mickey"
  x.last_name = "Mouse"
  x.date_of_birth = "May 15, 1928"
  x.save
end
```

The `destroy_all` method will delete _all_ of the records from a table, so be very careful with it! As you can see, I wrapped it within an `if` statement so that it will only happen in the development environment (our codespace), not the production environment (e.g. our server on Fly.io).

If you run the task now, you should see only 202 records in your table; the earlier ones we created are all gone. Also, notice that the IDs do not get reset; when a record is deleted, its ID number is "retired":

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1690651037/rails-db-9_eu28me.png)

Great! Now that we have a bunch of realistic records in our table, we're in good shape to practice searching, counting, sorting, etc.

Having realistic sample data is incredibly helpful while designing and developing an app, so good engineers invest some time in writing a sample data rake task.

In most of our projects going forward, I will include a sample data rake task for you. On your own projects, you'll have to write the sample data task for yourself!

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/contact-book/commit/cf33dce26106a2368ce9c91fb155196adf31ebc1)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/contact-book/tree/cf33dce26106a2368ce9c91fb155196adf31ebc1)

### Retrieving existing records

Now that we have shiny, realistic sample data to play with, let's open a `rails console` session and learn how to work with our records using our model.

#### .count

We can ask the `Contact` class how many records are in the table with the `.count` method:

```
[1] pry(main)> Contact.count
  Contact Count (0.4ms)  SELECT COUNT(*) FROM "contacts"
=> 202
```

#### ActiveRecord instance versus ActiveRecord Relation

We can retrieve all of the records in the table with the `.all` method:

```
[1] pry(main)> Contact.all
  Contact Count (0.8ms)  SELECT COUNT(*) FROM "contacts"
=> Contact::ActiveRecord_Relation (array with 202 Contact instances inside)
```

Notice the class of the return value: `Contact::ActiveRecord_Relation`.

An ActiveRecord Relation is the class that represents _a set of multiple records_ from the table (hence the name "relation").
{: .bg-red-100.py-1.px-5 }

ActiveRecord Relations are very similar to `Array`s. [Any method that you can call on an `Array`](https://learn.firstdraft.com/lessons/73), you can also call on a Relation; `.at`, `.each`, `.sample`, etc.

For example, let's save the relation containing all the records to a variable `x`, and then access the first element in the relation with `x.at(0)`:

```
[2] pry(main)> x = Contact.all
  Contact Count (0.5ms)  SELECT COUNT(*) FROM "contacts"
=> Contact::ActiveRecord_Relation (array with 202 Contact instances inside)

[3] pry(main)> x.at(0)
  Contact Load (1.0ms)  SELECT "contacts".* FROM "contacts"
=> #<Contact:0x00007f37f8064348
 id: 1,
 first_name: "Angila",
 last_name: "Russel",
 date_of_birth: Thu, 30 Jan 1913,
 street_address_1: "698 Ondricka Path",
 street_address_2: "Apt. 356",
 city: "Joeshire",
 state: "AR",
 zip: "02338",
 phone: "1-191-116-6902 x9912",
 notes: "Happiness can be found even in the darkest of times if only one remembers to turn on the light.",
 created_at: Sat, 29 Jul 2023 18:49:52.335131000 UTC +00:00>
```

(Your sample contacts will be different than mine, since we generated them with Faker.)

Each element within the relation is an instance of the `Contact` class that represents one record.
{: .bg-red-100.py-1.px-5 }

Let's look at the last element with `x.at(-1)`:

```
[4] pry(main)> x.at(-1)
=> #<Contact:0x00007f37f8506d60
 id: 200,
 first_name: "Jackqueline",
 last_name: "Bahringer",
 date_of_birth: Fri, 23 Feb 1912,
 street_address_1: "108 Cristopher Ports",
 street_address_2: "Apt. 572",
 city: "Port Evelinborough",
 state: "NE",
 zip: "96785",
 phone: "1-845-276-4377 x8122",
 notes: "It does not do to dwell on dreams and forget to live.",
 created_at: Sat, 29 Jul 2023 18:49:55.084958000 UTC +00:00>
```

Each of these objects are instances of the `Contact` class which represent different rows in our table.

Since we access the first and last elements of relations very often, there are convenience methods for accessing them:  `.first` (instead of `.at(0)`) and `.last` (instead of `.at(-1)`). You can use whichever one you prefer.

#### Attribute accessor methods

Let's pull out a random record with `Contact.all.sample` and store it in a variable `c`:

```
[5] pry(main)> c = Contact.all.sample
  Contact Load (0.8ms)  SELECT "contacts".* FROM "contacts"
=> #<Contact:0x00007f37f355a1e0
 id: 186,
 first_name: "Clyde",
 last_name: "Considine",
 date_of_birth: Thu, 08 Nov 1951,
 street_address_1: "318 Veola Manors",
 street_address_2: "Apt. 905",
 city: "North Loriannberg",
 state: "IN",
 zip: "87270",
 phone: "275-428-4275 x76972",
 notes: "Of course it is happening inside your head, Harry, but why on earth should that mean that it is not real?",
 created_at: Sat, 29 Jul 2023 18:49:54.894531000 UTC +00:00>
```

Once we have an individual instance of `Contact` stored in a variable, we can use the automatically-defined attribute accessor methods to retrieve the values stored in each column for that row:

```
[6] pry(main)> c.id
=> 186

[7] pry(main)> c.created_at
=> Sat, 29 Jul 2023 18:49:54.894531000 UTC +00:00

[8] pry(main)> c.first_name
=> "Clyde"

[9] pry(main)> c.last_name
=> "Considine"

[10] pry(main)> c.phone
=> "275-428-4275 x76972"
```

What happens if you try calling a method for a column that doesn't exist? For example, what if we try `.middle_name`?

```ruby
[11] pry(main)> c.middle_name
NoMethodError: undefined method `middle_name' for #<Contact id: 186, first_name: "Clyde", last_name: "Considine", date_of_birth: "1951-11-08", street_address_1: "318 Veola Manors", street_address_2: "Apt. 905", city: "North Loriannberg", state: "IN", zip: "87270", phone: "275-428-4275 x76972", notes: "Of course it is happening inside your head, Harry,...", created_at: "2023-07-29 18:49:54.894531000 +0000">
```

#### order

**Returns:** an ActiveRecord relation

The `.order` method lets you sort your collections by one or more columns. The argument to `.order` is a `Hash`, where the _key_ is the _column_ you want to sort by, and the _value_ is either `:asc` (for ascending order) or `:desc` (for descending order):

```ruby
Contact.all.order({ :last_name => :asc })
```

If you send `.order` a `Symbol` alone, outside of a `Hash`, then ascending order is assumed:

```ruby
Contact.all.order(:last_name)
```

To break ties, you can provide multiple columns in the `Hash`:

```ruby
Contact.all.order({ :last_name => :asc, :first_name => :asc, :date_of_birth => :desc })
```

This would first order by last name, then break ties using first name, then break ties using date of birth.

#### reverse_order

**Returns:** an ActiveRecord relation

`.reverse_order` reverses the ordering of a collection. Not particularly common, since you can use `:asc` and `:desc` to specify the direction you want explicitly, but there it is:

```ruby
Contact.order(:first_name).reverse_order
```

#### where

**Returns:** an ActiveRecord relation

Perhaps the most important READ method is `.where`. This is our bread-and-butter tool for _filtering_ a relation down using various criteria.

The argument to `.where` is a `Hash`, where the _key_ is the _column_ you want to filter by, and the _value_ is the criteria you want to filter by:

Let's say we wanted to look up all the records that have the value "Mouse" in the last_name column. We can do it using `.where` like this:

```ruby
x = Contact.all.where({ :last_name => "Mouse" })
x.count
```

A bit of syntactic sugar — to save us some typing, we can call `.where` directly on the class if we want to, rather than calling `.all` first:

```ruby
x = Contact.where({ :last_name => "Mouse" })
x.count
```

You can also provide multiple columns and values to filter by in the `Hash`:

```ruby
x = Contact.where({ :last_name => "Mouse", :first_name => "Minnie" })
x.count
```

#### where always returns a relation, never a single row

The return value from `.where` is always a Relation, regardless of how many results there are.
{: .bg-red-100.py-1.px-5 }

Whether there are 0, 1, or a million results, `.where` returns them within a Relation. What would you expect if you tried the following?

```ruby
x = Contact.where({ :id => 2 })
x.first_name
```

Think about it before you try it. Then, try it.

**We must use `.at(0)`** (a.k.a `.first`)  or `.at(-1)` (a.k.a. `.last`) or some other method **to take a row out of the relation** before we can use any of its instance methods. It doesn't make sense to ask the relation itself for e.g. `.first_name`; the relation is not an individual record.

```ruby
c = Contact.where({ :id => 2 }).at(0)
c.first_name
```

#### Using where with an array of criteria

**Returns:** an array of records

You can even use an `Array` in the argument to `.where`; it will then bring back the rows that match _any_ of the criteria for that column:

```ruby
Contact.where({ :last_name => ["Betina", "Woods"] })
```

#### Chaining wheres

**Returns:** an array of records

Since `.where` returns another relation, you can chain `.where`s one after the other:

```ruby
Contact.where({ :last_name => "Mouse" }).where({ :first_name => "Minnie" })
```

This _narrows_ the search.

### where(this).or(that)

**Returns:** an array of records

You can _broaden_ the search with `.or`:

```ruby
Contact.where({ :first_name => "Mickey" }).or(Contact.where({ :last_name => "Betina" }))
```

This may look a little funny. We tack `.or` onto the end of one collection, and the argument to `.or` is an _entire query_, starting from the class again. The return value is both collections merged together.

#### where.not(this)

**Returns:** an array of records

You can _negate_ a criteria with `.not`:

```ruby
Contact.where({ :last_name => "Mouse" }).where.not({ :first_name => "Mickey" })
```

You tack `where.not` on to a collection and it accepts all the same arguments as `.where`, but the result set is all of the records in the original collection _except_ the ones that match the criteria.

#### Where is everything

**Everything from looking up a movie's director to putting together a feed in a social network ultimately boils down to `.where`s and `.each`s.** I can't emphasize the importance of `.where` enough. Ask lots of questions.

### UPDATE

To update a row, first you need to locate it:

```ruby
c = Contact.where({ :id => 2 }).first
```

And then assign whatever new values you want to:

```ruby
c.first_name = "Minerva"
```

And then **don't forget to save the changes**:

```ruby
c.save
```

That's it.

## DELETE

To delete a row, first find it:

```ruby
c = Contact.where({ :id => 2 }).first
```

And then,

```ruby
c.destroy
```

Intense.

### Less commonly used queries

#### Fuzzy criteria

`.where` can also be used to search for partial matches by passing a fragment of SQL in a `String`, rather than passing a `Hash`:

```ruby
Contact.where("last_name LIKE ?", "%bet%")
```

The `?` in the first argument is a placeholder where the second argument, `"%bet%"`, gets inserted.

<aside markdown="1">
This is an advanced safety feature of Rails that prevents [SQL injection attacks](https://en.wikipedia.org/wiki/SQL_injection).
</aside>

The `%` characters are wildcards, which match anything in that position.

So that query would find all rows that have the fragment "bet" anywhere within the `last_name` column.

#### Less than or greater than

You can search for rows less than or greater than certain criteria:

```ruby
Contact.where("date_of_birth > ?", 30.years.ago)
Contact.where("last_name >= ? AND last_name <= ?", "A", "C")
```

Notice that you can have multiple placeholders `?` in the SQL fragment, and the subsequent arguments will be plugged in in order.

That last query, for a value within a range, can also be written with a `Hash` and a `Range`:

```ruby
Instructor.where({ :last_name => ("A".."C") })
```

This is particularly handy for searching for records within a particular range of times:

```ruby
start_date = 7.days.ago
end_date = Date.today
Contact.where({ :created_at => (start_date..end_date) })
```

With Ruby `Range`s, two dots means inclusive of the second value, and three dots means exclusive of the second value. E.g., `(1..4)` is 1, 2, 3, and 4; `(1...4)` is only 1, 2, and 3.

#### limit

**Returns:** an array of records

You can limit the number of records in a collection with `.limit`:

```ruby
Contact.where({ :last_name => "Mouse" }).limit(10)
```

This will return no more than 10 records.

#### offset

**Returns:** an array of records

You can skip some rows in the result set with `.offset`. This is useful for e.g. retrieving the second page of records, or choosing a random record:

```ruby
Contact.where({ :last_name => "Mouse" }).offset(10).limit(10)
```

#### maximum

**Returns:** a single value (of whatever datatype the column is)

You can calculate the largest/latest value in a particular column within a collection with `.maximum`:

```ruby
Contact.where({ :last_name => "Mouse" }).maximum(:date_of_birth)
```

#### minimum

**Returns:** a single value (of whatever datatype the column is)

You can calculate the smallest/oldest value in a particular column within a collection with `.minimum`:

```ruby
Contact.where({ :last_name => "Mouse" }).minimum(:date_of_birth)
```

#### average

**Returns:** a single value (`Integer` or `Float`)

You can calculate the average value in a particular column within a collection with `.average`:

```ruby
Review.where({ :venue_id => 4 }).average(:rating)
```

### Challenges

Given the above methods, try to answer the following questions about our data:

- Who is the oldest contact in the table?
- Who is the youngest contact in the table?
- How many contacts are in California (abbreviation: CA)?
- How many contacts have a last name starting with "G"?

## Must See Movies

In the next project, we're going to start building an app whose database we've already designed — Must See Movies.

In that project, the four tables we planned — movies, directors, characters, and actors — have already been created. I pre-created the tables to avoid e.g. typos in column names, which would make it hard for you to follow along.

Given an already-created database and tables, we'll learn how we can use the wonderful gem ActiveRecord to interact with those tables in Ruby. ActiveRecord translates our Ruby into highly performant, secure SQL automatically.


---

- Approximately how long (in minutes) did this lesson take you to complete?
{: .free_text_number #time_taken title="Time taken" points="1" answer="any" }

---
