---
title: "Basic Queries"
questions:
- "How do I write a basic query in SQL?"
objectives:
- "Write and build queries."
- "Filter data given various criteria."
- "Sort the results of a query."
keypoints:
- "It is useful to apply conventions when writing SQL queries to aid readability."
- "Use logical connectors such as AND or OR to create more complex queries."
- "Calculations using mathematical symbols can also be performed on SQL queries."
- "Adding comments in SQL helps keep complex queries understandable."
---

## Writing my first query

Let's start by using the **surveys** table. Here we have data on every
individual that was captured at the site, including when they were captured,
what plot they were captured on, their species ID, sex and weight in grams.

Let’s write an SQL query that selects only the year column from the
surveys table. SQL queries can be written in the box located under 
the "Execute SQL" tab. Click "Run SQL" to execute the query in the box, of type "Cmd + Return" on a Mac or "Ctrl + Return" on a Windows machine.

    SELECT year
    FROM surveys;

We have capitalized the words SELECT and FROM because they are SQL keywords.
SQL is case insensitive, but it helps for readability, and is good style.

When running queries from the command line or a script file, the queries must end in a semicolon. This is how the software knows it is at the end of the query. It's good practice to get in the habit of doing this, even though this interface does not require it.

If we want more information, we can just add a new column to the list of fields,
right after SELECT:

    SELECT year, month, day
    FROM surveys;

Or we can select all of the columns in a table using the wildcard *

    SELECT *
    FROM surveys;
    
    
> ## Challenge
>
> - What species were the animals they found and how mch did each of them weigh?
>
> > ## Solution
> > ~~~
> > SELECT species_id, weight
> > FROM surveys;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

### Limiting results

Sometimes you don't want to see all the results you just want to get a sense of
of what's being returned. In that case you can use the LIMIT command. In particular
you would want to do this if you were working with large databases.

    SELECT *
    FROM surveys
    LIMIT 10; 

### Unique values

If we want only the unique values so that we can quickly see what species have
been sampled we use `DISTINCT` 

    SELECT DISTINCT species_id
    FROM surveys;

If we select more than one column, then the distinct pairs of values are
returned

    SELECT DISTINCT year, species_id
    FROM surveys;

### Calculated values

We can also do calculations with the values in a query.
For example, if we wanted to look at the mass of each individual
on different dates, but we needed it in kg instead of g we would use

    SELECT year, month, day, weight/1000
    FROM surveys;

When we run the query, the expression `weight/1000.0` is evaluated for each row and appended to that row, in a new column. Note that because weight is an integer, if we divide by the integer 1000, the results will be reported as integers. In order to get more significant digits, you need to include the decimal point so that SQL knows you want the results reported as floating point numbers.

Expressions can use any fields, any arithmetic operators (+ - * /) and a variety of built-in functions (`MAX`, `MIN`, `AVG`, `SUM`, `ROUND`, `UPPER`, `LOWER`, `LEN`, etc). For example, we could round the values to make them easier to read.

    SELECT plot_id, species_id, sex, weight, ROUND(weight/1000, 2)
    FROM surveys;

> ## Challenge
>
> - Identify the species ID and weight in milligrams for each survey item and the date on which it was recorded.
>
> > ## Solution
> > ~~~
> > SELECT day, month, year, species_id, weight * 1000
> > FROM surveys;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

## Filtering

Databases can also filter data – selecting only the data meeting certain
criteria. For example, let’s say we only want data for the species
_Dipodomys merriami_, which has a species code of DM. We need to add a
`WHERE` clause to our query:

    SELECT *
    FROM surveys
    WHERE species_id='DM';

We can do the same thing with numbers.
Here, we only want the data since 2000:

    SELECT * FROM surveys
    WHERE year >= 2000;

We can use more sophisticated conditions by combining tests
with `AND` and `OR`. For example, suppose we want the data on *Dipodomys merriami*
starting in the year 2000, we can combine those filters using `AND`:

    SELECT *
    FROM surveys
    WHERE (year >= 2000) AND (species_id = 'DM');

Note that the parentheses are not needed, but again, they help with
readability. They also ensure that the computer combines `AND` and `OR`
in the way that we intend. (`AND` takes precedence over `OR` and will be evaluated before `OR`.)

If we wanted to get data for any of the *Dipodomys* species, which have
species codes `DM`, `DO`, and `DS`, we could combine the tests using OR:

    SELECT *
    FROM surveys
    WHERE (species_id = 'DM') OR (species_id = 'DO') OR (species_id = 'DS');
    
The above query is getting kind of long, so let's use a shortcut for all those `OR`s. This time, let’s use `IN` as one way to make the query easier to understand. `IN` is equivalent to saying `WHERE (species_id = "DM") OR (species_id = "DO") OR (species_id = "DS")`, but reads more neatly:

    SELECT *
    FROM surveys  
    WHERE species_id IN ("DM", "DO", "DS");

> ## Challenge
>
> - Produce a table listing the data for all individuals in Plot 1 
> that weighed more than 75 grams, telling us the date, species ID, and weight
> (in kg). 
>
> > ## Solution
> > ~~~
> > SELECT day, month, year, species_id, weight/1000
> > FROM surveys
> > WHERE (plot_id = 1) AND (weight > 75);
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

Now, if we wanted to get all the records from before 1980 or from 2000 or later that were about species DM, DS, or DO we would write it like this:

    SELECT *
    FROM surveys
    WHERE (year < 1980 OR year >=2000) AND (species_id IN ("DM", "DO", "DS"));

Because `AND` takes logical precendence over `OR` we need to be sure that we include parentheses so that the `OR` in the parentheses are evaluated before the `AND`.


We started with something simple, then added more clauses one by one, testing
their effects as we went along. For complex queries, this is a good strategy,
to make sure you are getting what you want. Sometimes it might help to take a
subset of the data that you can easily see in a temporary database to practice
your queries on before working on a larger or more complicated database.

> ## Challenge
>
> - Write a query that returns the year, month, day, species ID, and weight and plot ID for individuals caught in plots 1 or 2 and that weighed more than 75g. 
>
> > ## Solution
> > ~~~
> > SELECT day, month, year, species_id, weight, plot_id
> > FROM surveys
> > WHERE plot_id IN (1, 2) AND (weight > 75);
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}


## <a name="comment"></a> Commenting your queries

It is good practice to include comments in your queries so that you remember the purpose of the query and other people can understand it's purpose, and to describe what various parts of the query are doing or why they were included.  

Comments are ignored by the software and can be added in one of two ways.  

Short comments can be added by including them after two consecutive dashes. A line return signals the end of the comment. In the query below, the comment "-- only data from plots 1 & 2" in the 3rd line will be ignored.

    SELECT day, month, year, species_id, weight, plot_id
    FROM surveys
    WHERE plot_id IN (1, 2) -- only data from plots 1 & 2
        AND (weight > 75);

Longer comments can be added and separated from the query text by enclosing them in a forward slash and asterisk combination (`/*...*/`), as shown below. These comments can be written on multiple lines and the final asterisk-forward slash combination signals the end of the comment.  

    /*This query retrieves information about the date collected and species of all animals 
    collected in plots 1 & 2 that weighed over 75g */
    SELECT day, month, year, species_id, weight, plot_id
    FROM surveys
    WHERE plot_id IN (1, 2) -- only data from plots 1 & 2
        AND (weight > 75); -- only for those animals that weigh more than 75g

Details about commenting code can be found in the [SQLite documentation](https://www.sqlite.org/lang_comment.html).

## Sorting

We can also sort the results of our queries by using `ORDER BY`. Let's keep going with the query we've been writing.

    SELECT *
    FROM surveys
    WHERE (year < 1980 OR year >= 2000) AND (species_id IN ("DM", "DO", "DS"))
    ORDER BY plot_id ASC;

The keyword `ASC` tells us to order it in ascending order. We could alternately use `DESC` to get descending order.

    SELECT *
    FROM surveys
    WHERE (year < 1980 OR year >= 2000) AND (species_id IN ("DM", "DO", "DS"))
    ORDER BY plot_id DESC;

`ASC` is the default, so you don't actually need to specify it, but it helps with clarity.

We can also sort on several fields at once. Let's do by plot and then by species.

    SELECT *
    FROM surveys
    WHERE (year < 1980 OR year >= 2000) AND (species_id IN ("DM", "DO", "DS"))
    ORDER BY plot_id ASC, species_id DESC;

> ## Challenge
>
> - Alphabetize the **species** table by genus and then species.
>
> > ## Solution
> > ~~~
> > SELECT *
> > FROM species
> > ORDER BY genus, species;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

## Order of execution

Another note for ordering. We don’t actually have to display a column to sort by it. For example, let’s say we want to order by the plot ID and species ID, but we only want to see the date, plot, and weight information.

    SELECT day, month, year, plot_id, weight
    FROM surveys
    WHERE (year < 1980 OR year >= 2000) AND (species_id IN ("DM", "DO", "DS"))
    ORDER BY plot_id ASC, species_id DESC;

We can do this because sorting occurs earlier in the computational pipeline than field selection.

The computer is basically doing this:

1. Collecting data from tables according to `FROM`
2. Filtering rows according to `WHERE`
3. Sorting results according to `ORDER BY`
4. Displaying requested columns or expressions according to `SELECT`


When we write queries, SQL dictates the query parts be supplied in a particular order: `SELECT`, `FROM`, `JOIN...ON`, `WHERE`, `GROUP BY`, `ORDER BY`. Note that this is not the same order in which the query is executed. (We'll get to `JOIN...ON` and `GROUP BY` in a bit.)


> ## Challenge
>
> - Let's try to combine what we've learned so far in a single
> query. Using the surveys table write a query to display the three date fields,
> `species_id`, and weight in kilograms (rounded to two decimal places), for
> individuals captured in 1999, ordered alphabetically by the `species_id`.
> - Write the query as a single line, then put each clause on its own line, and
> see how more legible the query becomes!
>
> > ## Solution
> > ~~~
> > SELECT year, month, day, species_id, ROUND(weight / 1000, 2)
> > FROM surveys
> > WHERE year = 1999
> > ORDER BY species_id;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}
