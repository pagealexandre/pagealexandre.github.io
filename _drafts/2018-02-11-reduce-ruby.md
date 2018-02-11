---
layout: post
title: Ruby - reduce / inject function
permalink: reduce.html
categories: ruby
---

## Reduce function

The reduce function reduce an array into a single value. It iterates through the array while having a accumulator variable (memo). This variable contain the return value of the block given as parameter for each elem of the array. The block contain as param the element itself and the accumulator value.


{% highlight ruby %}
[1] pry(main)> a = [1, 2, 3, 4, 5]
=> [1, 2, 3, 4, 5]
[2] pry(main)> a.reduce {|elem, accumulator| accumulator += elem}
=> 15
{% endhighlight %}

Here we simply add all the element of the array into the accumulator variable.
So on the first loop we have :

* accumulator = 1 (It takes the first value of the array if we don't specify one)
* elem = 2
* 2 + 1

Then for the second round of loop :

* accumulator = 3
* elem = 3
* 3 + 3 = 6

And so on.

The true idea with reduce is that the accumulator variable (memo) is reassigned each time in the loop by the operation we specify in the block.

{% highlight ruby %}
a = [2, 3, 4]
=> [2, 3, 4]
[4] pry(main)> a.reduce {|elem, accumulator| accumulator * elem}
=> 24
{% endhighlight %}

* accumulator = 2
* elem = 3
* 2 * 3 = 6
* accumulator = 6

Then
* accumulator = 6
* elem = 4
* 6 * 4 = 24
* accumulator = 24

Then it return the accumulator (memo) variable.

One interesting thing is that the reduce function can take as an argument a default value for the accumulator variable. If not specified then, the function will take the first value of the array as the accumulator variable and directly begin with the second value of the array to execute the block.

We can pass a default accumulator variable, here 0, like this :
{% highlight ruby %}
a = [2,3,4]
a.reduce(0) {|elem, sum| sum * elem} # 24
{% endhighlight %}

## The reduce function with symbol

The reduce function can also take a symbol. It will iterates over the array and apply an operation.

{% highlight ruby %}
[5] pry(main)> a = [2, 3, 4]
=> [2, 3, 4]
[6] pry(main)> a.reduce(:+)
=> 9
{% endhighlight %}

It's exactly the same as writing `a.reduce {|elem, sum| sum += elem}`. But I guess it's more readable to use a symbol.


## Recoding reduce 🧙‍

If you read my previous article about map, there's not really any magic here : we call a proc on each elem and pass as argument the elem itself and the accumulator variable.

A basic implementation of reduce look like this :

{% highlight ruby %}
def reduce(array, &block)
  accumulator = 0
  array.each do |elem|
    accumulator = block.call(accumulator, elem)
  end
  accumulator
end
{% endhighlight %}

Again the only thing that change from the each method is that we pass a second variable to the proc (`accumulator`) which is then reassigned as the return value of the proc.

Let's now look at a more robust version of reduce, which can handle symbol using the open class technique on the Array class.

{% highlight ruby %}
class Array

  def reduce(accumulator = nil, operation = nil, &block)
    if operation && block
      raise ArgumentError
    end

    if operation.nil? && block.nil?
      operation = accumulator
      accumulator = nil
    end

    block = begin
      case operation
      when Symbol
        lambda { |acc, value| acc.send(operation, value) }
      when nil
        block
      end
    end

    if accumulator.nil?
      accumulator = first
      skip_first = true
    end

    index = 0

    self.each do |elem|
      unless skip_first && index == 0
        accumulator = block.call(accumulator, elem)
      end
      index += 1
    end
    accumulator
  end
end
{% endhighlight %}

The `operation` variable contain a symbol, it is optionnal. The `accumulator` is the memo value which stock the return value of the proc at each round of the loop. This is this value that the
function will return.

**Two important things** : the behavior of the function make a case on the operation variable to either execute a the given block or the symbol.

`acc.send(operation, value)` is where the magic happen with the symbol as parameter. This snippets will be called on each iteration of the loop to increment the accumulator value.

For example with
`[1, 2, 3, 4, 5].reduce(:+)`
This is what going on on the first round of loop :


* accumulator = 1

* operation = `:+`
* value = 2
* `1.send(:+, 2)` will return 3

Then on the second round of loop :

* accumulator = 3
* operation = `:+`
* value = 3
* `3.send(:+, 3)` will return 6

And so on.

The second important thing is the `skip_first` and `index` variable. Basically, it sum up the fact that if you don't specify any default accumulator value, then `accumulator` will take the first value of the array (`accumulator = first`) and we begin the loop with the second value of the array. We skip the first round of the loop.

We reach the end of this articles about the reduce function.

As a conclusion, the `reduce` function and the `inject` are aliases according to the [ruby doc](The inject and reduce methods are aliases. There is no performance benefit to either) :
> The inject and reduce methods are aliases. There is no performance benefit to either.

Sources :

- [https://mauricio.github.io/2015/01/12/implementing-enumerable-in-ruby.html](https://mauricio.github.io/2015/01/12/implementing-enumerable-in-ruby.html)
- [https://ruby-doc.org/core-2.5.0/Enumerable.html#method-i-reduce](https://ruby-doc.org/core-2.5.0/Enumerable.html#method-i-reduce)
