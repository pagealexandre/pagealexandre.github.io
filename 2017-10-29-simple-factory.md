<!-- ---
layout: post
title: Swift - Design Pattern - Factory
permalink: factory-pattern.html
categories: ios swift design pattern
---


## Introduction

A factory is an object which has the role to create multiple object from the same family, same type. In other word this is a wrapper that we go through to instantiate different object from the same place. (This is why most of the time factory are actually singleton).  Since we are going to show this through Swift, let me take an example of Factory producing iPhone.

Here are the different available iPhone at the moment :
* iPhone X
* iPhone 8

Let's create an enum for all of these models :


{% highlight swift %}
enum iPhone {
  case X
  case iPhone8
}
{% endhighlight %}

I created the following class to represent the different iPhone.

class iPhone {
  var screenSize: Double?
}

class iPhoneX: iPhone {
  
  override init() {
    super.init()
    self.screenSize = 5.8
  }
}

class iPhone8: iPhone {

  override init() {
    super.init()
    self.screenSize = 4.7
  }
}
{% endhighlight %}

This is the factory which has the role to create different instance of iPhone.lol

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

{% endhighlight %} -->