---
layout: post
title: Swift - Design Pattern - Factory
permalink: factory-pattern.html
categories: ios swift design pattern
---


## Introduction

A factory is an object which has the role to create multiple objects from the same family, same type. In other word this is a wrapper that we go through to instantiate different objects from the same place. Since we are going to show this through Swift, let me take an example of Factory producing iPhones.

Here are the different available iPhone at the moment :
* iPhone X
* iPhone 8

Let's create an enum for all of these models :

{% highlight swift %}
enum Models {
  case X
  case iPhone8
}
{% endhighlight %}

These are the classes representing the different iPhones on the market. We use inheritance to save the common attributes.
{% highlight swift %}
class iPhone {
  var screenSize: Double
  
  init(screenSize: Double) {
    self.screenSize = screenSize
  }
}

class iPhoneX: iPhone {
  
  init() {
    super.init(screenSize: 5.8)
  }
}

class iPhone8: iPhone {

  init() {
    super.init(screenSize: 4.7)
  }
}
{% endhighlight %}

This is the factory which has the role to create different instance of iPhone.

{% highlight swift %}
class iPhoneFactory {

  static func produceIphone(type: Model) -> iPhone {
    switch type {
      case .iPhone8:
        return iPhone8()
      case .X :
        return iPhoneX()
    }
  }

}

var instance = iPhoneFactory.produceIphone(type: .iPhone8)
print(type(of: instance)) // iPhone8 
print(instance.screenSize) // 4.7

{% endhighlight %}

To recap basically the Factory is a wrapper which allow to instantiante different class type based on parameters, here, an enum.

