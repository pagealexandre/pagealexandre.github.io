---
layout: post
title: Ruby - All you need to know about scope
permalink: ruby-scope.html
categories: ruby
---


## Introduction

In this post, I will talk about scope in ruby. We will see the following :

* Top Level Scope
* Scope gate
* Flate scope

## Top Level Scope
_kesako_ ? In order to understand the concept there is one prerequisite to understand first.
In ruby everything is an object, meaning that when you launch irb or pry, you're already inside an object.

__pry__
{% highlight ruby %}
[1] pry(main)> self
=> main
[2] pry(main)> self.class
=> Object
{% endhighlight %}

You understand it, even when you launch the interpreter you are already in an object called main belonging to the Object class. 


The top level scope simply put is the scope you're in when you don't have called any methods yet or they just finished and you just arrived in the interpreter.

__example.rb__
{% highlight ruby %}
str = "abc" # Top Level Scope

class A
  # I am not in the top level scope
  def b
  # I am not in the top level scope
  end
end

# Top Level Scope
{% endhighlight %}

**From the moment you're in a class, you're not in the top level scope anymore**

## Scope Gate

Whenever the ruby interpreter meet the following keywords : `class`, `def`, `module`, it leaves the previous scope and open a new one. Each of the keywords act as a scope Gate : it determines which local variables are accessible at any given time.

__scope_gate.rb__
{% highlight ruby %}
a = 1
class MyClass # Scope Gate entering new scope
  b = 2
  def my_method # Scope Gate entering new scope
    c = 3
    local_variables # => [:c]
  end           # Scope Gate leaving scope
end           # Scope Gate leaving scope
{% endhighlight %}

In this case none of the defined variable are in the same scope, so none of them can use the other to compute something.

# Flat Scope
Now that we understand the Scope gate concept, let's see how we can counter it. Instead of using the keyword `class` we can use `Class.new` and in this way we will be able to access variable from the top level scope.

{% highlight ruby %}
[9] pry(main)> a = 'bingo'
=> "bingo"
[10] pry(main)> Class.new do
[10] pry(main)*   puts a
[10] pry(main)* end
bingo
=> #<Class:0x007ff2a8806a90>
{% endhighlight %}

The same idea can be applied for method.

{% highlight ruby %}
[13] pry(main)> class A
[13] pry(main)*   b = 1
[13] pry(main)*
[13] pry(main)*   define_method :c do
[13] pry(main)*     puts b
[13] pry(main)*   end
[13] pry(main)* end
=> :c
[14] pry(main)> A.new.c
1 # Bingo
=> nil
{% endhighlight %}

Theses snippets of code rely on the scope of block in ruby, indeed creating a block capture the local binding, in the case : `b`

## Conclusion

The top level scope and the Scope gate are two important concept in ruby, however I have doubts about the usefulness of Flat scope, since it does not respect at all OOP.
