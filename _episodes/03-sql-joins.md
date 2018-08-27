---
title: "Joins"
questions:
- "How do I bring data together from separate tables?"
objectives:
- "Employ joins to combine data from two tables."
- "Apply functions to manipulate individual values."
- "Employ aliases to assign new names to tables and columns in a query."
keypoints:
- "Use the `JOIN` command to combine data from two tables---the `ON` or `USING` keywords specify which columns link the tables."
- "Regular `JOIN` returns only matching rows. Other join commands provide different behavior, e.g., `LEFT JOIN` retains all rows of the table on the left side of the command."
- "All of the commands we have discussed can be run from the command line as well."
---

## Joins

To combine data from two tables we use the SQL `JOIN` command, which comes after
the `FROM` command.

We will also need to use the keyword `ON` to tell the computer which columns provide the link ([Primary Key > Foreign Key](#design)) between the two tables. In this case, the species_id column in the **species** table is defined as the primary key. It contains the same data as the **survey** table's species_id column, which is the foreign key in this case. We want to join the tables on these species_id fields.

    SELECT *
    FROM surveys
    JOIN species
    ON surveys.species_id = species.species_id;

`ON` is like `WHERE`, it filters things out according to a test condition. We use
the `table.colname` format to tell the manager what column in which table we are
referring to.

Field names that are identical in both tables can confuse the software so you must specify which table you are talking about, any time you mention these fields. You do this by inserting the table name in front of the field name as `table.colname`, as I have done above in the `SELECT` and `GROUP BY` parts of the query.

We often won't want all of the fields from both tables, so anywhere we would
have used a field name in a non-join query, we can use `table.colname`.

For example, what if we wanted information on when individuals of each
species were captured, but instead of their species ID we wanted their
actual species names.

    SELECT surveys.year, surveys.month, surveys.day, species.genus, species.species
    FROM surveys
    JOIN species
    ON surveys.species_id = species.species_id;


> ## Challenge:
>
> - Identify the genus, species, and weight of every individual captured at the site
>
> > ## Solution
> > ~~~
> > SELECT species.genus, species.species, surveys.weight
> > FROM surveys
> > JOIN species
> > ON surveys.species_id = species.species_id;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

### Different join types

We can count the number of records returned by our original join query.

    SELECT COUNT(*)
    FROM surveys
    JOIN species
    USING (species_id);

Notice that this number is smaller than the number of records present in the
survey data.

This is because, by default, SQL only returns records where the joining value
is present in the joined columns of both tables (i.e. it takes the _intersection_
of the two join columns). This joining behavior is known as an `INNER JOIN`.
In fact the `JOIN` command is simply shorthand for `INNER JOIN` and the two
terms can be used interchangably as they will produce the same result.

We can also tell the computer that we wish to keep all the records in the first
table by using the command `LEFT OUTER JOIN`, or `LEFT JOIN` for short.

> ## Challenge:
>
> - Re-write the original query to keep all the entries present in the `surveys`
> table. How many records are returned by this query?
>
> > ## Solution
> > ~~~
> > SELECT * FROM surveys
> > LEFT JOIN species
> > USING (species_id);
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

### Combining joins with sorting and aggregation

Joins can be combined with sorting, filtering, and aggregation. So, if we
wanted average mass of the individuals on each different type of treatment, we
could do something like

    SELECT plots.plot_type, AVG(surveys.weight)
    FROM surveys
    JOIN plots
    ON surveys.plot_id = plots.plot_id
    GROUP BY plots.plot_type;

> ## Challenge:
>
> - How many animals of each genus were caught in each plot? Report the answer with the greatest number at the top of the list. 
>
> > ## Solution
> > ~~~
> > SELECT surveys.plot_id, species.genus, COUNT(*) AS number_indiv
> > FROM surveys
> > JOIN species
> > ON surveys.species_id = species.species_id
> > GROUP BY species.genus, surveys.plot_id
> > ORDER BY number_indiv DESC;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

You can also combine many tables using a `JOIN`. The query must include enough `JOIN`...`ON` clauses to link all of the tables together. In the query below, we are now looking at the count of each species for each type of plot during each year. This required 1) adding in an extra `JOIN`...`ON` clause, 2) including plot_type in the `SELECT` portion of the statement, and 3) adding plot_type to the `GROUP BY` function:

    SELECT surveys.species_id, surveys.year, plots.plot_type, COUNT(*), AVG(weight)
    FROM surveys
    JOIN species ON surveys.species_id = species.species_id
    JOIN plots ON plots.plot_id = surveys.plot_id
    GROUP BY surveys.species_id, year, plot_type
    ORDER BY COUNT(*) DESC;
    
To practice we have some optional challenges for you.

> ## Challenge (optional):
>
> SQL queries help us ask specific questions which we want to answer about our data. The real skill with SQL is to know how to translate our scientific questions into a sensible SQL query (and subsequently visualize and interpret our results).
>
> Have a look at the following questions; these questions are written in plain English. Can you translate them to *SQL queries* and give a suitable answer?  
> 
> 1. How many plots from each type are there?  
> 
> 2. How many specimens are of each sex are there for each year?  
> 
> 3. How many specimens of each species were captured in each type of plot?  
> 
> 4. What is the average weight of each taxa?  
> 
> 5. What are the minimum, maximum and average weight for each species of Rodent?  
>
> 6. What is the average hindfoot length for male and female rodent of each species? Is there a Male / Female difference?  
> 
> 7. What is the average weight of each rodent species over the course of the years? Is there any noticeable trend for any of the species?  
>
> > ## Proposed solutions:
> >
> > 1. Solution: 
> > ~~~
> > SELECT plot_type, COUNT(*) AS num_plots
> > FROM plots
> > GROUP BY plot_type;
> > ~~~
> > {: .sql}
> >
> > 2. Solution:
> > ~~~
> > SELECT year, sex, COUNT(*) AS num_animal
> > FROM surveys
> > WHERE sex IS NOT NULL
> > GROUP BY sex, year;
> > ~~~
> > {: .sql}
> >
> > 3. Solution: 
> > ~~~
> > SELECT species_id, plot_type, COUNT(*) 
> > FROM surveys 
> > JOIN plots USING(plot_id) 
> > WHERE species_id IS NOT NULL 
> > GROUP BY species_id, plot_type;
> > ~~~
> > {: .sql}
> >
> > 4. Solution:
> > ~~~
> > SELECT taxa, AVG(weight) 
> > FROM surveys 
> > JOIN species ON species.species_id = surveys.species_id
> > GROUP BY taxa;
> > ~~~
> > {: .sql}
> >
> > 5. Solution:
> > ~~~
> > SELECT surveys.species_id, MIN(weight), MAX(weight), AVG(weight) FROM surveys 
> > JOIN species ON surveys.species_id = species.species_id 
> > WHERE taxa = 'Rodent' 
> > GROUP BY surveys.species_id;
> > ~~~
> > {: .sql}
> >
> > 6. Solution:
> > ~~~
> > SELECT surveys.species_id, sex, AVG(hindfoot_length)
> > FROM surveys JOIN species ON surveys.species_id = species.species_id 
> > WHERE (taxa = 'Rodent') AND (sex IS NOT NULL) 
> > GROUP BY surveys.species_id, sex;
> > ~~~
> > {: .sql}
> >
> > 7. Solution:
> > ~~~
> > SELECT surveys.species_id, year, AVG(weight) as mean_weight
> > FROM surveys 
> > JOIN species ON surveys.species_id = species.species_id 
> > WHERE taxa = 'Rodent' GROUP BY surveys.species_id, year;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

## <a name="command"></a> Running sqlite3 from the command line

The SQLite Manager plug in we have been using for this lesson is a convenient interface for working with a database. However, you might want to store and edit queries in a file, or be able to automatically direct your output to a file. These are tasks are much simpler when run from the command line.

SQLite3 can be run directly from the command line. Detailed information about how to do this is available at [sqlite.org](http://sqlite.org/cli.html).

In order to do this you need to be in the shell (Terminal window on Mac), and you need to be in the directory where your .sqlite database file resides. Then just type `sqlite3` followed by the name of the .sqlite databases file and you are instantly in the sql interface, ready to type in a query.

Type `.help` at the prompt to see options available, or check out the more detailed explanation of most of your choices at the sqlite.org link above.   
Type `.tables` to see a list of all your database tables.    
Type any `SELECT` statement directly from the prompt. Be sure to end with a semicolon. That's how the computer knows you are at the end of the query. Hit `Enter` or `Return` after the semicolon to run the query.  
Run a query from a .sql file (your query saved as a text document with a .sql file extension) by typing `.read` followed by a space and the file name.  
Type `.mode csv` to change the standard output to .csv format (instead of delimited by pipes).    
Type `.once` followed by a space and a file name to send the output from the next `.read` query to a file instead of to the screen.  
Type `.exit` to leave sqlite.  
