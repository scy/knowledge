# Database Design and Peculiarities

## In General

### Using "key-set pagination" instead of `OFFSET`

Markus Winand explains in [Paging Through Results](https://use-the-index-luke.com/sql/partial-results/fetch-next-page) an alternative to `SELECT … LIMIT 10 OFFSET 10`: 
Pagination by modifying the `WHERE` clause to simply skip all results that the previous page has returned.

This requires a deterministic `ORDER BY` and is a bit more complex. 
Also, you can't simply jump to arbitrary pages. 
However, in return you get significantly better speed (for higher pages) and stable results: 
While the results `OFFSET` returns change based when something new is inserted to (or deleted from) the results on previous pages, his method stays stable.

## MySQL

(And MariaDB, I guess.)

### Comparing Strings

When comparing strings with `=`, it sometimes looks like MySQL is removing (or trimming) spaces on the right side of the string, i.e. `'a' = 'a  '`. 
This can also lead to the impression that MySQL does a trim on string values when storing them in the database. 
Strings that differ in space padding only will also cause duplicate key errors, even though they _look_ different – and in fact, they are.

[According to Stack Overflow](https://stackoverflow.com/a/10495807), the real reason is that `=` is defined in SQL-92 and SQL:2008 with this characteristic:

> If the length in characters of X is not equal to the length in characters of Y, then the shorter string is effectively replaced, for the purposes of comparison, with a copy of itself that has been extended to the length of the longer string by concatenation on the right of one or more pad characters

The default pad characters are spaces.

You can work around this by using `LIKE` (which doesn't pad the strings) or apparently by using `binary 'a' = 'a  '`.
