---
layout: post
title: Swift - Optional, binding, chaining and cast ?!
permalink: optional-and-cast.html
categories: ios swift
---

{: .intro }
When I began to learn Swift, I was confuse with the ! and ? symbols used everywhere for cast or even variables. This articles describes everything related to these symbols : what do they mean and how to use it.

## Optional
An optional is a sort of wrapper that allow to deal with the absence of value in a variable. This is because Swift has been made to privilege safety and prevent error during the execution of an application. An optional of type Int e.g is represented the following way: 

{% highlight swift %}
let number: Int?
{% endhighlight %}

The interrogative point indicates the variable might contain a value but it might have no value at all.

To access the value of an optional we use the exclamation mark (!).This technique is called the *forced unwrapping*.  Accessing a nil value with ! will trigger a runtime error, so it's alway smart to check the optional before accessing it. 

{% highlight swift %}
if number != nil {
	print(number!)
}

{% endhighlight %}


## Optional Binding

Optional binding allows to check if the variable actually contains a value through an if statement :

{% highlight swift %}
if let b = Int(a) {
	print(b)
}
{% endhighlight %}

If the optional Int returned by Int(a) contains a value, then the value is assigned to b. This is why I don't need to use the ! inside the condition.


## Implicitly unwrapped optionals
When we are sure the value of a variable will not be nil, e.g constant with let, we can use *implicitly unwrapped optionals* to directly unwrap the value during variable declaration.

{% highlight swift %}
let version: String! = "1.0" // Implicitly unwrapped optionals
let str: String? = "1.0"

print(version) // "1.0"
print(str!) // "1.0"

{% endhighlight %}

## Optional Chaining
The optional chaining concept proposes to query properties, methods, values that might be in an optional and thus might be nil. If the optional have a value, then the call to the property, method, value succeeds.

- The optional chaining return nil if variable is nil.

{% highlight swift %}
class Person {
	var car: Car?
}

class Car {
	let numberOfWheels = 4
}

let alex = Person()

let wheels = alex.car!.numberOfWheels // trigger a runtime error
let wheels = alex.car?.numberOfWheels // return nil

{% endhighlight %}

The idea here is to uses the question mark instead of the exclamation mark. The optional chaining on line 12 will always return an optional : Int? or if the car is nil (which is the case here), then it return nil. However if the car is not empty :

{% highlight swift %}
alex.car = Car()
let wheels = alex.car?.numberOfWheels // 4
{% endhighlight %}


## As, As? and As!

To avoid repetitive content for the next example, we will consider these two classes :
{% highlight swift %}
class Animal {}

class Dog: Animal {}

{% endhighlight %}

The as operator is used for cast, specially **upcast** i.e conversion of a specialized class to its superclass. Basic example are :

{% highlight swift %}
let a = 5.0 as Double
let dog: Dog = Dog()
let animal: Animal = dog as Animal // Upcast
{% endhighlight %}

But the real usefulness of the as symbol relate when doing **downcast** i.e conversion of a superclass to specialized one :
{% highlight swift %}

let animal: Animal = Dog()

animal as Dog // will raise an error

animal as! Dog // Force downcast
animal as? Dog

{% endhighlight %}

as! will force the cast, so we have to be sure of the type (Will crash at runtime otherwise). However as? will perform the cast if possible, otherwise it will simply return nil. It makes a lot of sense to use it that way :

{% highlight swift %}
if let animal = animal as? Dog {
	print("BINGO")
}
{% endhighlight %}

- To recap *as* is used for normal cast, or upcast.
- as! and as? are used for downcast : a conversion from a superclass to a specialized class.
- On the first hand using as! force the cast and thus, we need be sure of the type otherwise a runtime error will appear.
- On the other hand, using as? will return nil if something fail.


