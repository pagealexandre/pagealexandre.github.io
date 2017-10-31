---
layout: post
title: Swift - Design Pattern - Factory Method
permalink: factory-method.html
categories: ios swift
---


## Simple Factory reminder

Remember the [Factory Pattern]({% post_url 2017-10-29-simple-factory %}) ? Well, the thing is Apple and the iPhone aren't the result of only the iPhone 8 and iPhone X. For example, with the iPhone we've got iPhone 4, iPhone 5, iPhone 6 etc ... The idea with Factory Method is to use multiples Factories for different object.

We add factory that will defer instantiation of object to subclass. One of the thing I struggle to understand is that a Factory can instantiate different type of iPhone (Product), e.g for iPhone 7 Factory it make sens to create multiple different object like iPhone 7, 7Plus.

![factory-method]({{"/assets/factory-method.png"}})

## Actors
When talking about Factory Method we've got different players:
* __Product__, this is the iPhone class in our example. It is the parent class that every object will be based on. *Be careful* I choose to use a class, but this can also be an interface (or protocol in Swift)
* __Concrete Product__ : This is the iPhoneSE and iPhone7* class, this is the concret implementation of the object. It inherit from the iPhone parent class.
* __Creator__ : This is an interface where the Factory Method will be specified. In our example this is the Factory Protocol
* __Concrete Creator__ : The concret protocol is a class implementing the Creator protocol. In our example this is the iPhone5factory for example.
* __Client__ : I often found on the internet schemas with the client, this is the object that will invoke the Concret Product. In the example below, this is represented by me instantiating the object

## Implementation

Ok ! Now that you've seen the UML, let's implement it :

__Types.swift__
{% highlight swift %}
enum Types {
  case Regular
  case Plus
}
{% endhighlight %}

__iPhones.swift__
{% highlight swift %} 
class iPhone {
  let screenSize: Double
  
  init(screenSize: Double) {
    self.screenSize = screenSize
  }
}

class iPhoneSE: iPhone {
  init() {
    super.init(screenSize: 4.0)
  }
}

class iPhone7: iPhone {
  init() {
    super.init(screenSize: 4.7)
  }
}

class iPhone7Plus: iPhone {
  init() {
    super.init(screenSize: 5.5)
  }
}
{% endhighlight %}

__Factories.swift__
{% highlight swift %}
protocol Factory {
  func produce(type: Types) -> iPhone
}

class iPhoneSEFactory: Factory {
  func produce(type: Types) -> iPhone {
    return iPhoneSE()
  }
}

class iPhone7Factory: Factory {
  func produce(type: Types) -> iPhone {
    switch type {
      case .Regular:
        return iPhone7()
      case .Plus:
        return iPhone7Plus()
    }
  }
}
{% endhighlight %}

__extension.swift__
{% highlight swift %}
extension Factory {
  func produce(type: Types = .Regular) -> iPhone {
    return produce(type: type)
  }
}
{% endhighlight %}

All right, just an explanation on the extension. This snippet of code is proper to Swift, I am trying to add default parameter value to the produce function in the Factory protocol. This will allow me to call the produce function of iPhoneSEFactory without giving any parameters. Also in the future, if I wanted to add another iPhone and Factory then it will be more clear for another developer to ignore this parameter if not needed instead of instantiating the object with an enum which is ignored at the end. I would found this confusing.

__main.swift__
{% highlight swift %}

var factory: Factory = iPhone7Factory()
var iphone: iPhone = factory.produce(type: .Plus) // FactoryMethod.iPhone7Plus

print(String(describing: iphone.self))

factory = iPhoneSEFactory()
iphone = factory.produce() // FactoryMethod.iPhoneSE

print(String(describing: iphone.self))


{% endhighlight %}

