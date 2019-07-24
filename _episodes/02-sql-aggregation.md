---
title: "SQL Aggregation and aliases"
questions:
- "How can I summarize my data by aggregating, filtering, or ordering query results?"
- "How can I make sure column names from my queries make sense and aren't too long?"
objectives:
- "Apply aggregation to group records in SQL."
- "Filter and order results of a query based on aggregate functions."
- "Employ aliases to assign new names to items in a query."
keypoints:
- "Use the `GROUP BY` keyword to aggregate data."
- "Functions like `MIN`, `MAX`, `AVERAGE`, `SUM`, `COUNT`, etc. operate on aggregated data."
- "Aliases can help shorten long queries. To write clear and readible queries, use the `AS` keyword when creating aliases."
- "Use the `HAVING` keyword to filter on aggregate properties."
---

## COUNT and GROUP BY

Aggregation allows us to combine results by grouping records based on value, also it is useful for
calculating combined values in groups.

Let’s go to the surveys table and find out how many individuals there are.
Using the wildcard * simply counts the number of records (rows):

    SELECT COUNT(*)
    FROM surveys;

There are many other aggregate functions included in SQL including `SUM`, `MAX`, `MIN`, and `AVG`. We can find out the average of weight for all of those individuals by using the function `AVG`.

    SELECT COUNT(*), AVG(weight)
    FROM surveys;

Something a little more useful might be finding the number and average weight of a particular species in our survey, like DM.

	SELECT COUNT(*), AVG(weight)
    FROM surveys
    WHERE species_id="DM";

> ## Challenge
>
> Write a query that returns: the total weight, average weight, minimum and maximum weights
> for all animals caught over the duration of the survey.
> Can you modify it so that it outputs these values only for weights between 5 and 10?
>
> > ## Solution
> > ~~~
> > -- All animals
> > SELECT SUM(weight), AVG(weight), MIN(weight), MAX(weight)
> > FROM surveys;
> >
> > -- Only weights between 5 and 10
> > SELECT SUM(weight), AVG(weight), MIN(weight), MAX(weight)
> > FROM surveys
> > WHERE (weight > 5) AND (weight < 10);
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

Now, let's see how many individuals were counted in each species. We do this
using a `GROUP BY` clause

    SELECT species_id, COUNT(*)
    FROM surveys
    GROUP BY species_id;

`GROUP BY` tells SQL what field or fields we want to use to aggregate the data.
If we want to group by multiple fields, we give `GROUP BY` a comma separated list.

> ## Challenge
>
> 1. Identify how many animals were counted in each year in total
> 2. Identify how many animals were counted each year per species
> 3. Determine the average weight of each species in each year
>
> Now try to combine the queries in 2 & 3 to list how many and the average weight for each species in each year. 
>
> > ## Solution of 1
> > ~~~
> > SELECT year, COUNT(*)
> > FROM surveys
> > GROUP BY year;
> > ~~~
> > {: .sql}
> {: .solution}
> >
> > ## Solution of 2
> > ~~~
> > SELECT year, species_id, COUNT(species_id)
> > FROM surveys
> > GROUP BY year, species_id;
> > ~~~
> > {: .sql}
> {: .solution}
> >
> > ## Solution of 3
> > ~~~
> > SELECT year, species_id, AVG(weight)
> > FROM surveys
> > GROUP BY year, species_id;
> > ~~~
> > {: .sql}
> {: .solution}
>
> > ## Solution of 2 and 3
> > ~~~
> > SELECT year, species_id, COUNT(*), AVG(weight) 
> > FROM surveys
> > GROUP BY year, species_id;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

## Ordering Aggregated Results

We can order the results of our aggregation by a specific column, including
the aggregated column. Let’s count the number of individuals of each
species captured, ordered by the count, then by species_id:

    SELECT species_id, COUNT(*)
    FROM surveys
    GROUP BY species_id
    ORDER BY COUNT(*), species_id;

## Aliases

As queries get more complex names can get long and unwieldy. To help make things
clearer we can use aliases to assign new names to things in the query.

We can use aliases in column names or table names using `AS`:

    SELECT MAX(year) AS last_surveyed_year
    FROM surveys;

The `AS` isn't technically required, so you could do

    SELECT MAX(year) last_surveyed_year
    FROM surveys;

but using `AS` is much clearer so it is good style to include it.

## The `HAVING` keyword

In the previous episode, we have seen the keyword `WHERE`, allowing to
filter the results according to some criteria. SQL offers a mechanism to
filter the results based on **aggregate functions**, through the `HAVING` keyword.

For example, we can request to only return information
about species with a count higher than 10:

    SELECT species_id, COUNT(species_id)
    FROM surveys
    GROUP BY species_id
    HAVING COUNT(species_id) > 10;

The `HAVING` keyword works exactly like the `WHERE` keyword, but uses
aggregate functions instead of database fields to filter.

If you use `AS` in your query to rename a column, `HAVING` you can use this
information to make the query more readable. For example, in the above
query, we can call the `COUNT(species_id)` by another name, like
`occurrences`. This can be written this way:

    SELECT species_id, COUNT(species_id) AS occurrences
    FROM surveys
    GROUP BY species_id
    HAVING occurrences > 10;

Note that in both queries, `HAVING` comes *after* `GROUP BY`. One way to
think about this is: the data are retrieved (`SELECT`), which can be filtered
(`WHERE`), then joined in groups (`GROUP BY`); finally, we can filter again based on some
of these groups (`HAVING`).

> ## Challenge
>
> Write a query that returns, from the `species` table, the number of
> `species` in each `taxa`, only for the `taxa` with more than 10 `species`.
>
> > ## Solution
> > ~~~
> > SELECT taxa, COUNT(*) AS n
> > FROM species
> > GROUP BY taxa
> > HAVING n > 10;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}
