---
layout: post
title: Ruby - Active Record - Query optimization
permalink: orm-optimization-1.html
categories: ruby orm active record
---

### All vs find_each

## All
Immagine you have a database with a table `users` with one hundred thousand records. If you use the `all` method of active records, basically what will happen is the ORM will run the following query :

{% highlight sql %}
SELECT "users".* FROM "users"
{% endhighlight %}

Just so you know it took 3.5ms to execute this query for 2000 users. The second thing you have to know is that the memory storing these users took 96 bytes, which is okay for 2000 users, but what about 500 000 posts of theses users ? (500 000 * 96) / 2000 = 0.22mo. You can reduce the memory needed for this query by using batches in the `find_each` function. The garbadge collector will then clean the memory while you process other records.

## find_each
Let's take a look at rails source code in order to analyze the behavior of the function. Basically, find_each has two behaviors based on wether we pass a block or not.

Here is the source code :
{% highlight ruby %}
def find_each(options = {})
  if block_given?
    find_in_batches(options) do |records|
      records.each { |record| yield record }
    end
  else
    enum_for :find_each, options do
      options[:start] ? where(table[primary_key].gteq(options[:start])).size : size
    end
  end
end
{% endhighlight %}

If we don't pass any block, the code will return a [`enumerator` which will iterate by calling `find_each (again)`](https://ruby-doc.org/core-2.5.1/Object.html#method-i-enum_for). when you call each on the enumerator. The block is given to compute the size of the enumerator lazyly.

Otherwise if a block is given, then we call `find_in_batches`. Basically calling `find_in_batches` yield an array of records (a batch). The `find_each` use this function to iterate over each records of the batches and yield for each. In other words, when you call `find_each` you get as parameter one records of the batch while using `find_in_batches` the parameters contain an array of records i.e a entire batch.

Let's quickly look at the implementation of `find_in_batches` :

{% highlight ruby %}
def find_in_batches(options = {})
  options.assert_valid_keys(:start, :batch_size)

  relation = self
  start = options[:start]
  batch_size = options[:batch_size] || 1000

  unless block_given?
    return to_enum(:find_in_batches, options) do
      total = start ? where(table[primary_key].gteq(start)).size : size
      (total - 1).div(batch_size) + 1
    end
  end

  if logger && (arel.orders.present? || arel.taken.present?)
    logger.warn("Scoped order and limit are ignored, it's forced to be batch order and batch size")
  end

  relation = relation.reorder(batch_order).limit(batch_size)
  records = start ? relation.where(table[primary_key].gteq(start)).to_a : relation.to_a

  while records.any?
    records_size = records.size
    primary_key_offset = records.last.id
    raise "Primary key not included in the custom select clause" unless primary_key_offset

    yield records

    break if records_size < batch_size

    records = relation.where(table[primary_key].gt(primary_key_offset)).to_a
  end
end
{% endhighlight %}

We find again the same logic when we don't pass any block to the function : return an enumerator and set the size using a SQL query: total / batch_size.

According to [this post](http://www.monkeyandcrow.com/blog/reading_rails_how_do_batched_queries_work/) and my experience digging in the code what happen is first we set `LIMIT` and `ORDER` parameter on the SQL queries that will be made. This is done through this code : 

```
relation = relation.reorder(batch_order).limit(batch_size)
```

`batch_size` is set from the options you gave or default to 1000
`batch_order` is using reflection to set the SQL query : 

```
def batch_order
  "#{quoted_table_name}.#{quoted_primary_key} ASC"
end
```

All of this result in have a query like this one : 

```
Users Load (1.8ms)  SELECT  "users".* FROM "users" WHERE ("users"."id" > 2000) ORDER BY "users"."id" ASC LIMIT $1  [["LIMIT", 2000]]
```
We see the ORDER BY "users"."id" ASC which come from the previous method and set the limit. By doing that we assure that users id will be fetch in the good order (asc) so that we can fetch all the records by batch.

Finally for the loop condition what happen is active records fetch the records by batches. The first time through the `start` variable, the second times using the id of the last records fetched in the batch. It then yield the records.

If the number of records fetched is inferior to the number of records in the batch, it mean that we fetched all the records and we are done. We break the loop.

Funnily enough if this is equal we can still be done. If I have 2000 users and ask for a batches of 1000 users. `find_each` will do 3 query
the one from 0 - 1000; then 1000 - 2000; and 2000 - 3000. The last one will result in no records and we will break the loop.


## Conclusion
- When we don't pass any block to `find_each` or `find_in_batches` those will return an enumerator. It will contain the size of records. (Nice because we don't need to iterate over it to recompute it again)
- `find_in_batches` yields with an array of results i.e the batch. `find_each` yield for each batch that `find_in_batches` yields. Nice implementation.
- `find_in_batches` (and thus `find_each` too) use `ORDER BY` and `LIMIT` to get an ascending order of the records and select them only by batch (using `LIMIT` keywords). For this it select the last id of the previously fetch records and fetch the new batch of records by getting those superior to this id.

From what I computed using `all` will not take as much memory as I thought however, we can easily end up in a situation where the call to this function can take 3-4 seconds (`SELECT * FROM myTable`), in this case it might be better to use `find_each` (SELECT * FROM mytable ORDER BY fields.id ASC LIMIT myBatch) because of the segregation of the records by batch, basically querying less to be faster.

It's funny to see that the variable pass to `find_each` are the one used for the SQL query.

