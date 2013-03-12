# Expression Builder

The expression builder API is designed to allow programmatic generation of queries. It mimics the API of other well
known builder domains, such as SQL. It additionally escapes all user input and makes sure you don’t accidentally
introduce query injections. While the query API of Solr or ElasticSearch is read only and injections will not lead to
data destruction, you could still leak private information (such as products not yet published).

## Simple queries

Constructing simple queries is quite easy. Given that you have configured a default search field or pass one with the
query, all you need to do is to call the `eq()` method. Let’s construct a simple query that consists of a single phrase
`"this is my search term"`.

```php
$query = $eb->eq('this is my search term');
```

Next step might be to limit those searches to a specific field (`myField:"this is my search term"`).

```php
$query = $eb->field('myField', $eb->eq('this is my search term'));
```

A popular use case is to fall back on everything. To support that, expression builder has an `all()` method that returns
a `*:*` if all values passed to it are empty. `$query1` will contain `field:"value"` while `$query2` contain `*:*`.

```php
$query1 = $eb->all($eb->field('field', $eb->eq('value')));
$query2 = $eb->all($eb->field('field', $eb->eq(null)));
```

*Note:* the expression builder API generally assumes `null` to be an empty query.


### Requiring and prohibiting expressions

To create queries that require a certain condition to be true, we need to add a plus sign (`+`) in front of it. Let’s
create a query in the form of `+field:"search term"`.

```php
$eb->req($eb->field('field', $eb->eq('search term')));
```

Let’s negate that expression and make sure we only get documents where `"search term`" is not present (`-field:"search
term"`).

```php
$eb->prhb($eb->field('field', $eb->eq('search term')));
```

If we need to require/prohibit based on dynamic input, we can also use the `bool()` function. Passing `true` as an
operator will require, passing `false` will prohibit an expression.

```php
$eb->bool($eb->field('field', $eb->eq('search term')), true);
$eb->bool($eb->field('field', $eb->eq('search term')), false);
```

### Constructing range queries

Another powerful feature of Solr is range queries. You can search between numbers, letters and dates. Let’s start with a
simple numeric range query to get only products with ratings between 5 and 10 (`rating:[5 TO 10]`).

```php
$eb->field('rating', $eb->range(5, 10));
```

If we want to use an open upper bound (`rating:[5 TO *]`), we can do that as well:

```php
$eb->field('rating', $eb->range(5, null));
```

`[` is used to define implicit ranges, '{' is used for explicit ranges. We can either use `btwnRange()` or pass `false`
as a third parameter to `range()` (`rating:{5 TO 10}`).

```php
$eb->field('rating', $eb->btwnRange(5, 10));
```

We promised date ranges, let’s create a query that searches for products from 2012 (`publishedOn:[2012-01-01T00:00:00Z
TO 2012-12-31T23:59:56]`).

```php
$eb->field('publishedOn', $eb->dateRange(new DateTime('2012-01-01 00:00:00'), new DateTime('2012-12-31 23:59:59')));
```