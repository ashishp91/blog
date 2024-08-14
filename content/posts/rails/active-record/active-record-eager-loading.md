---
title: "Eager Loading"
date: 2024-04-11T11:16:36+05:30
draft: false
tags: ["Ruby", "Rails", "ActiveRecord"]
---

### What is Eager Loading ?

> Eager loading is a mechanism through which ActiveRecord loads the associated records in memory, reducing the number of executed sql queries.

There are following ways to do eager loading in ActiveRecord:

1. includes
2. preload
3. eager_load

### Why do we need Eager Loading ?

Let's say we've to get the author's last name of some books. For that the query will look like:

```ruby
books = Book.limit(10)
books.each do |book|
  puts book.author.last_name
end
```

Running this queries executes following sql queries:

```sql
  Book Load (0.2ms)  SELECT "books".* FROM "books" LIMIT $1  [["LIMIT", 10]]
  Author Load (0.2ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 5], ["LIMIT", 1]]
Crona
  Author Load (0.1ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
Schamberger
  Author Load (0.1ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 3], ["LIMIT", 1]]
Schamberger
  Author Load (0.1ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 4], ["LIMIT", 1]]
Friesen
  Author Load (0.1ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" = $1 LIMIT $2  [["id", 2], ["LIMIT", 1]]
Williamson
```

This executes 11 queries in total, 1 for loading the books and the 10 for loading the authors as we loop through them one by one.
This is classic proble called `N+1 Queries Problem` where in order to load N associations of a record, it executes N+1 queries.

How to solve this ?
Well, using eager loading fixes the problem. Let's see how.

### Method 1 - Using includes

When using `includes`, ActiveRecord ensures that all of the specified associations are loaded in the minimum possible queries. Let's take our books example and use includes in that:

```ruby
books = Book.includes(:author).limit(10)

books.each do |book|
  puts book.author.last_name
end
```

On executing the above code, it only takes 2 queries:

```sql
SELECT "books".* FROM "books" LIMIT $1  [["LIMIT", 10]]
SELECT "authors".* FROM "authors" WHERE "authors"."id" IN ($1, $2, $3, $4)  [["id", 5], ["id", 3], ["id", 4], ["id", 2]]
```

One more thing to note with `includes` is that when the query has conditions on associations, `includes` will use `left_outer_join` instead of individual queries. This is because the conditions can be applied on the `left_outer_join`, let's see an example:

```ruby
books = Book.includes(:author).where(author: { last_name: 'Rowling' }).limit(10)

books.each do |book|
  puts book.author.last_name
end
```

This code when executed produces following queries:

```sql
SELECT "books"."id" AS t0_r0, "books"."title" AS t0_r1, "books"."year_published" AS t0_r2, "books"."isbn" AS t0_r3, "books"."price" AS t0_r4, "books"."views" AS t0_r5, "books"."author_id" AS t0_r6, "books"."supplier_id" AS t0_r7, "books"."created_at" AS t0_r8, "books"."updated_at" AS t0_r9, "author"."id" AS t1_r0, "author"."first_name" AS t1_r1, "author"."last_name" AS t1_r2, "author"."title" AS t1_r3, "author"."created_at" AS t1_r4, "author"."updated_at" AS t1_r5
FROM "books" LEFT OUTER JOIN "authors" "author" ON "author"."id" = "books"."author_id" WHERE "author"."last_name" = $1 LIMIT $2  [["last_name", "Rowling"], ["LIMIT", 10]]
```

As we can see that there is only one `left_outer_join` query generated.

### Method 2 - Using preload

When using `preload`, ActiveRecord ensures that each association is loaded using one query per association. Here's how our book example will work with `preload`:

```ruby
books = Book.preload(:author).limit(10)
books.each do |book|
  puts book.author.last_name
end
```

And executing that will look something like this:

```sql
  Book Load (0.2ms)  SELECT "books".* FROM "books" LIMIT $1  [["LIMIT", 10]]
  Author Load (0.2ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" IN ($1, $2, $3, $4)  [["id", 5], ["id", 3], ["id", 4], ["id", 2]]
Crona
Schamberger
Schamberger
Friesen
Williamson
```

Here we're using 2 queries
- 1st query is for loading the books records
- 2nd query is to load the associated author records

> `preload` requires M+1 queries where the ActiveRecord object has to load M associations.

For eg. if our `books` example required to load `authors` and `reviews` then it'd take 2+1=3 queries with `preload`, here is how it looks like:

```ruby
books = Book.preload(:author, :reviews).limit(10)
  Book Load (0.3ms)  SELECT "books".* FROM "books" /* loading for pp */ LIMIT $1  [["LIMIT", 10]]
  Author Load (0.2ms)  SELECT "authors".* FROM "authors" WHERE "authors"."id" IN ($1, $2, $3, $4)  [["id", 5], ["id", 3], ["id", 4], ["id", 2]]
  Review Load (0.2ms)  SELECT "reviews".* FROM "reviews" WHERE "reviews"."book_id" IN ($1, $2, $3, $4, $5)  [["book_id", 1], ["book_id", 2], ["book_id", 3], ["book_id", 4], ["book_id", 5]]
```

It's interesting to note that `preload` doesn't work when there is a condition on an association. Here's an example:

```ruby
books = Book.preload(:author, :reviews).where(author: { last_name: 'Rowling' })
An error occurred when inspecting the object: #<ActiveRecord::StatementInvalid:"PG::UndefinedTable: ERROR:  missing FROM-clause entry for table \"author\"\nLINE 1: SELECT \"books\".* FROM \"books\" WHERE \"author\".\"last_name\" = $...\n                                            ^\n">
...
```

This because `left_outer_join` is required to make conditions work with associations. As `preload` doesn't use `left_outer_join` it throws an error.

### Method 3 - Using eager_load

When using `eager_load`, ActiveRecord uses `left_outer_join` to load all the associations. Let's say we need to get the reviews of each book this time:

```ruby
books = Book.eager_load(:reviews).limit(10);

books.each do |book|
  puts book.reviews.sum(&:last_name)
end
```

And executing this uses following queries:

```sql
SELECT DISTINCT "books"."id" FROM "books" LEFT OUTER JOIN "reviews" ON "reviews"."book_id" = "books"."id" LIMIT $1  [["LIMIT", 10]]
SELECT "books"."id" AS t0_r0, "books"."title" AS t0_r1, "books"."year_published" AS t0_r2, "books"."isbn" AS t0_r3, "books"."price" AS t0_r4, "books"."views" AS t0_r5, "books"."author_id" AS t0_r6, "books"."supplier_id" AS t0_r7, "books"."created_at" AS t0_r8, "books"."updated_at" AS t0_r9, "reviews"."id" AS t1_r0, "reviews"."title" AS t1_r1, "reviews"."body" AS t1_r2, "reviews"."rating" AS t1_r3, "reviews"."state" AS t1_r4, "reviews"."customer_id" AS t1_r5, "reviews"."book_id" AS t1_r6, "reviews"."created_at" AS t1_r7, "reviews"."updated_at" AS t1_r8 FROM "books" LEFT OUTER JOIN "reviews" ON "reviews"."book_id" = "books"."id" WHERE "books"."id" IN ($1, $2, $3, $4, $5)
Williamson
Schamberger
Schamberger
Friesen
Crona
 =>
```

It uses only 2 queries as opposed to 11 queries.
