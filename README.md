# Joins

What we did in the previous section was run subqueries (a query inside of a query) and use that result as a restriction on what we were searching for. Can you imagine running this kind of query against 5 tables?! Madness. Also, what if you want to get all of the rows from both tables in one row?

SQL supports a feature called 'joins'. Joins are really cool! It's a way to query against multiple tables and get all the information from those tables at once. This way we can say "Tenser has these powers and this is how much damage they take: ..." with one query.

SQLite has a bunch of ways to help you join data together. The first one we'll look at it the Inner Join

## Inner Join

We want to get a list of all wizards and their powers. We'll Start out with some familiar syntax

``` sql
SELECT *
FROM wizards
```

Now The `JOIN` magic. The `JOIN` keyword takes a second table:
``` sql
SELECT *
FROM wizards
INNER JOIN powers
```

Nothing... You may be thinking, "What else would we need? It's obviously going to look for the wizard_id on powers." Unfortunately, that's not the case. There's not set database convention on what to name foreign keys so we'll have to specify this ourselves using the `ON` keyword:

``` sql
SELECT *
FROM wizards
INNER JOIN powers
ON wizards.id = powers.wizard_id;
```

Here we specify that the wizards' id column matches the powers' wizard_id column.

Our result:
```
sqlite> SELECT *
   ...> FROM wizards
   ...> INNER JOIN powers
   ...> ON wizards.id = powers.wizard_id;

 Alviarin | 103 | White |  1 |  9 | fire ball      |      7 |         1
 Eriadna  |  85 | Green |  3 | 10 | ice storm      |     13 |         3
 Bigby    |  40 | Green |  4 | 11 | shape shift    |      0 |         4
 Tenser   |  75 | Brown |  2 | 12 | invisibility   |      0 |         2
 Alviarin | 103 | White |  1 | 13 | lightning bolt |     23 |         1
 Tenser   |  75 | Brown |  2 | 14 | earth quake    |     15 |         2
 Bigby    |  40 | Green |  4 | 15 | tidal wave     |     44 |         4
 Eriadna  |  85 | Green |  3 | 16 | teleport       |      0 |         3
```

There are a few things to discuss before we move on.

### Selecting rows to display

Our previous result looks messy with all those ids and stuff flying around. We really just want to return the wizard's name and age, along with the power's name and damage. Also, we want to see the columns and headers. Here's how we do it:

```
sqlite> .mode column
sqlite> .header on
sqlite> SELECT wizards.name, wizards.age, powers.name, powers.damage
   ...> FROM wizards
   ...> INNER JOIN powers
   ...> ON wizards.id = powers.wizard_id;

   name   | age |      name      | damage
--+----------------+--------
 Alviarin | 103 | fire ball      |      7
 Eriadna  |  85 | ice storm      |     13
 Bigby    |  40 | shape shift    |      0
 Tenser   |  75 | invisibility   |      0
 Alviarin | 103 | lightning bolt |     23
 Tenser   |  75 | earth quake    |     15
 Bigby    |  40 | tidal wave     |     44
 Eriadna  |  85 | teleport       |      0
```

Cool! If you're joining a table, you can use its name in your `SELECT` statement to reference its columns!  We can do better though. Notice how there are 2 'names' on this output? Let's set the column name for the power name to just be 'power' using th `AS` keyword:

```
sqlite> SELECT wizards.name, wizards.age, powers.name AS power, powers.damage
   ...> FROM wizards
   ...> INNER JOIN powers
   ...> ON wizards.id = powers.wizard_id;

   name   | age |     power      | damage
--+----------------+--------
 Alviarin | 103 | fire ball      |      7
 Eriadna  |  85 | ice storm      |     13
 Bigby    |  40 | shape shift    |      0
 Tenser   |  75 | invisibility   |      0
 Alviarin | 103 | lightning bolt |     23
 Tenser   |  75 | earth quake    |     15
 Bigby    |  40 | tidal wave     |     44
 Eriadna  |  85 | teleport       |      0
```

## Inner Join exclusion

For this next section, let's create a wizard but not give him any powers:

```
INSERT INTO wizards (name, age, color) VALUES ('Steven', 30, 'Green');
```

Then run our previous join:

```
sqlite> SELECT *
   ...> FROM wizards
   ...> INNER JOIN powers
   ...> ON wizards.id = powers.wizard_id;

   name   | age | color | id | id |      name      | damage | wizard_id
--+-------+----+----+----------------+--------+-----------
 Alviarin | 103 | White |  1 |  9 | fire ball      |      7 |         1
 Eriadna  |  85 | Green |  3 | 10 | ice storm      |     13 |         3
 Bigby    |  40 | Green |  4 | 11 | shape shift    |      0 |         4
 Tenser   |  75 | Brown |  2 | 12 | invisibility   |      0 |         2
 Alviarin | 103 | White |  1 | 13 | lightning bolt |     23 |         1
 Tenser   |  75 | Brown |  2 | 14 | earth quake    |     15 |         2
 Bigby    |  40 | Green |  4 | 15 | tidal wave     |     44 |         4
 Eriadna  |  85 | Green |  3 | 16 | teleport       |      0 |         3

```

What the heck!? Where'd Steven go?! Here is what makes the Inner Join different from other joins. The `ON` statement tells SQLite to ONLY return rows from both tables where there's a match. To get access to our new untrained wizard, we'll need the help of another JOIN. The OUTER JOIN.

## OUTER JOINS

If you try to change the word INNER for OUTER in our query, you'll get honking error:

```
sqlite> SELECT *
   ...> FROM wizards
   ...> OUTER JOIN powers
   ...> ON wizards.id = powers.wizard_id;
Error: RIGHT and FULL OUTER JOINs are not currently supported
```

An outer join requires a bit more information. To understand these options, we'll have to teach you how to tell your left from your right.

### LEFT VS RIGHT
In every join, there's a left table and a right table. If you're expecting the `ON` statement to tell you, THINK AGAIN! In our example, the 'left' table is the first table mentioned, so the 'wizards' table is the left table, making the 'powers' table the right table. Back to OUTER JOINs

### ALL THE WIZARDS!

To get our wizards out including the powerless We'll have to run this query:

```
SELECT *
FROM wizards
LEFT OUTER JOIN powers
ON wizards.id = powers.wizard_id;
```

And now we see that "Steven" shows up even though he's powerless:

```

   name   | age | color | id | id |      name      | damage | wizard_id
--+-------+----+----+----------------+--------+-----------
 Alviarin | 103 | White |  1 |  9 | fire ball      |      7 |         1
 Eriadna  |  85 | Green |  3 | 10 | ice storm      |     13 |         3
 Bigby    |  40 | Green |  4 | 11 | shape shift    |      0 |         4
 Tenser   |  75 | Brown |  2 | 12 | invisibility   |      0 |         2
 Alviarin | 103 | White |  1 | 13 | lightning bolt |     23 |         1
 Tenser   |  75 | Brown |  2 | 14 | earth quake    |     15 |         2
 Bigby    |  40 | Green |  4 | 15 | tidal wave     |     44 |         4
 Eriadna  |  85 | Green |  3 | 16 | teleport       |      0 |         3
 Steven   |  30 | Green |  5 |    |                |        |
```

SQLite doesn't currently support RIGHT OUTER JOINs or FULL OUTER JOINs, but we'll go over it so you can see how it works in other Databases, like Postgres.

Let's add a power that no wizard wants:

```
INSERT INTO powers (name, damage) VALUES ('awkwardly asking for the time', 10);
```

Now let's try that RIGHT OUTER JOIN:

```
SELECT *
FROM wizards
RIGHT OUTER JOIN powers
ON wizards.id = powers.wizard_id;
```

Our Result:

```
   name   | age | color | id | id |             name              | damage | wizard_id
--+-------+----+----+-------------------------------+--------+-----------
 Alviarin | 103 | White |  1 |  9 | fire ball                     |      7 |         1
 Eriadna  |  85 | Green |  3 | 10 | ice storm                     |     13 |         3
 Bigby    |  40 | Green |  4 | 11 | shape shift                   |      0 |         4
 Tenser   |  75 | Brown |  2 | 12 | invisibility                  |      0 |         2
 Alviarin | 103 | White |  1 | 13 | lightning bolt                |     23 |         1
 Tenser   |  75 | Brown |  2 | 14 | earth quake                   |     15 |         2
 Bigby    |  40 | Green |  4 | 15 | tidal wave                    |     44 |         4
 Eriadna  |  85 | Green |  3 | 16 | teleport                      |      0 |         3
          |     |       |    | 17 | awkwardly asking for the time |     10 |
(9 rows)
```

What if we want to see all of the rows that match, even if they don't pair up? That's when a `FULL OUTER JOIN` comes in:

```
SELECT *
FROM wizards  
FULL OUTER JOIN powers
ON wizards.id = powers.wizard_id;
   name   | age | color | id | id |             name              | damage | wizard_id
--+-------+----+----+-------------------------------+--------+-----------
 Alviarin | 103 | White |  1 |  9 | fire ball                     |      7 |         1
 Eriadna  |  85 | Green |  3 | 10 | ice storm                     |     13 |         3
 Bigby    |  40 | Green |  4 | 11 | shape shift                   |      0 |         4
 Tenser   |  75 | Brown |  2 | 12 | invisibility                  |      0 |         2
 Alviarin | 103 | White |  1 | 13 | lightning bolt                |     23 |         1
 Tenser   |  75 | Brown |  2 | 14 | earth quake                   |     15 |         2
 Bigby    |  40 | Green |  4 | 15 | tidal wave                    |     44 |         4
 Eriadna  |  85 | Green |  3 | 16 | teleport                      |      0 |         3
          |     |       |    | 17 | awkwardly asking for the time |     10 |
 Steven   |  30 | Green |  5 |    |                               |        |
(10 rows)
```

## Wizard Factions
**Back to SQLite**

Eventually small groups break out intent on world domination, and we need a way to track them. Our wizards aren't dummies, so they join multiple factions to hedge their bets.

Let's create a `factions` table to track wizard membership. Using what we know now, we might think to create it using something like this:

```
CREATE TABLE factions(
  name TEXT,
  agenda TEXT,
  wizard_id INTEGER
);
```

But this won't work. If Alviarin joins the "Free Puppies" faction, and the "Guns for Ferrets" faction all works according to plan, but what if Bigby joins the "Free Puppies" faction? How do we track that? We'd need to duplicate the "Free Puppies" agenda just to add Bigby. If they change their name to "Sort of Free Puppies", We'd have to find all of the rows and update them. There's another way.

First we create the `factions` table, with no mention of the wizards or foreign keys:

```
CREATE TABLE factions(
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT,
  agenda TEXT
);
```

THEN we create a table who's purpose is to link factions to wizards. This is called a join table, since that's what it's used for. This table will have 2 columns, one for the wizard_id and one for the faction_id. It will not need an id field of its own.

```
CREATE TABLE factions_wizards(
  faction_id INTEGER,
  wizard_id INTEGER
);
```

Next let's add a couple of factions:

```
INSERT INTO factions(name, agenda) VALUES ('Free Puppies', 'Winning hearts and minds through free puppies'), ('Guns for Ferrets', 'No explanation needed');
```

Now let's add Alviarin to both of these groups:

```
INSERT INTO factions_wizards (wizard_id, faction_id) VALUES (1, 1);
INSERT INTO factions_wizards (wizard_id, faction_id) VALUES (1, 2);
```

Now let's add Tenser to the 'Free Puppies' faction:

```
INSERT INTO factions_wizards (wizard_id, faction_id) VALUES (2, 1);
```

Finally, with all of this set up, let's create a query that finds all wizards that are members of the "Free Puppies" faction. We'll build this one up:

```
sqlite> SELECT *
   ...> FROM wizards
   ...> INNER JOIN factions_wizards
   ...> ON wizards.id = factions_wizards.wizard_id
```

Here we are linking the wizards table and the factions_wizards table on the id and wizard_id properties respectively. This is only half of what we want though. This query will only return the wizards and a table that has faction_ids. We want factions! To do this, we'll need another INNER JOIN:

```
SELECT *
FROM wizards
INNER JOIN factions_wizards
ON wizards.id = factions_wizards.wizard_id
INNER JOIN factions
ON factions.id = factions_wizards.faction_id
```

If we run this, we'll get ALL of the factions joined with their wizards. Not bad! We want to only get members of the Free Puppies faction. To do this, we'll employ something we used earlier: A `WHERE` clause.

```
SELECT *
FROM wizards
INNER JOIN factions_wizards  
ON wizards.id = factions_wizards.wizard_id  
INNER JOIN factions
ON factions.id = factions_wizards.faction_id
WHERE factions.name = 'Free Puppies';
```

Our result looks like this:

```
   name   | age | color | id | wizard_id | faction_id | id |     name     |                    agenda
--+-------+----+-----------+------------+----+--------------+-----------------------------------------------
 Alviarin | 103 | White |  1 |         1 |          1 |  1 | Free Puppies | Winning hearts and minds through free puppies
 Tenser   |  75 | Brown |  2 |         2 |          1 |  1 | Free Puppies | Winning hearts and minds through free puppies

```

### LEFT VS RIGHT REDUX
We know that in the above query `wizards` is the LEFT and `factions_wizards` is the RIGHT, but what is `factions`? The RIGHT RIGHT? We can test this by creating a Faction on one joins. Let's add one, then run our previous query without the `WHERE` clause.

```
INSERT INTO factions (name, agenda) VALUES ('Muggle Lovers', 'They need love too');
```

Then run the query:

```
SELECT *
FROM wizards
INNER JOIN factions_wizards
ON wizards.id = factions_wizards.wizard_id
INNER JOIN factions
ON factions.id = factions_wizards.faction_id;
name   | age | color | id | wizard_id | faction_id | id |       name       |                    agenda
--+-------+----+-----------+------------+----+------------------+-----------------------------------------------
 Alviarin | 103 | White |  1 |         1 |          1 |  1 | Free Puppies     | Winning hearts and minds through free puppies
 Alviarin | 103 | White |  1 |         1 |          2 |  2 | Guns for Ferrets | No explanation needed
 Tenser   |  75 | Brown |  2 |         2 |          1 |  1 | Free Puppies     | Winning hearts and minds through free puppies
(3 rows)

```

No Muggle Lovers as expected, since an inner join only returns records where all of the `JOIN` `ON` clauses match.

But what if we wanted to find all Factions that had no members?  First, let's try to figure out how to return all of the factions along with their users. Each JOIN statement defines what is the RIGHT side of the join. By saying `JOIN factions`, we're setting `factions` is the right side of this join, leaving `factions_wizards` to be the left.

Knowing this, let's leave everything the same, except we'll change the final JOIN to be a RIGHT OUTER JOIN so it includes all of the rows from the factions table, even if it doesn't have a match.

```
SELECT *
FROM wizards
INNER JOIN factions_wizards
ON wizards.id = factions_wizards.wizard_id
RIGHT OUTER JOIN factions
ON factions.id = factions_wizards.faction_id;
```

WOOT! Our result looks like this:

```
  name   | age | color | id | wizard_id | faction_id | id |       name       |                    agenda
--+-------+----+-----------+------------+----+------------------+-----------------------------------------------
 Tenser   |  75 | Brown |  2 |         2 |          1 |  1 | Free Puppies     | Winning hearts and minds through free puppies
 Alviarin | 103 | White |  1 |         1 |          1 |  1 | Free Puppies     | Winning hearts and minds through free puppies
 Alviarin | 103 | White |  1 |         1 |          2 |  2 | Guns for Ferrets | No explanation needed
          |     |       |    |           |            |  3 | NAMWLA           | They need love too
(4 rows)
```

And our Final part is to only show factions with no members. Here we'll use a `WHERE` clause to search for rows where the wizard name is NULL (SQL for empty) using `WHERE wizards.name IS NULL`:


```
SELECT *
FROM wizards
LEFT OUTER JOIN factions_wizards
ON wizards.id = factions_wizards.wizard_id
LEFT OUTER JOIN factions
ON factions.id = factions_wizards.faction_id
WHERE factions_wizards.wizard_id IS NULL;
```

WORKS!

## Resources
* [A Visual Explanation of SQL Joins](http://blog.codinghorror.com/a-visual-explanation-of-sql-joins/)
* [The 3 Ring Binder Model](http://blog.seldomatt.com/blog/2012/10/17/about-sql-joins-the-3-ring-binder-model/)

* [SQL Innter Join](http://www.padjo.org/tutorials/database-joins/sql-inner-join/)
* [Joining on Many To Many Relationships](http://www.padjo.org/tutorials/database-joins/sql-many-to-one/)
* [SQL Left Joins](http://www.padjo.org/tutorials/database-joins/sql-left-joins/)

<a href='https://learn.co/lessons/sql-book-joins' data-visibility='hidden'>View this lesson on Learn.co</a>
