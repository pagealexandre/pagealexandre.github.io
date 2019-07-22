---
layout: post
title: Virtual attributes
permalink: virtual-attributes.html
categories: ruby rails
---

I recently face an issue in my job where I was suppose to create a resource, N times. A first approach is to use the default `params` object used in controller to send a variable containing the number of time the object should be created. However, since the creation of the object should be done through another model, a second approach could be to use virtual attributes. The idea behind a virtual attributes is that this is an accessor / attributes of the model which is not persistant in the database. We could then fill it with a form.

Just before going in the code here is the scenario: we are going to create an object `EpcOrder`, which should create N times another object `GtinEpcOrder`.

## Step 1, the form.
__form.html.erb__
{% highlight ruby %}
<%= f.input :gtins_count, as: :integer %>
<%= f.input :description, as: :text %>
<%= f.submit %>
{% endhighlight %}

The first thing to do is create a form to fill the model. Here the description is a persistant attributes of the model, while `gtins_count` is the virtual attributes, we call it that way but really there's no convention, what matter is that the name has to match with the vitual attributes defined in the model. (See just below)

## Step 2, the model.
__epc_order.rb__
```rb
class EpcOrder < ApplicationRecord
  validates :gtins_count, numericality: { only_integer: true, greater_than: 0, less_than_or_equal_to: 10.000 }
  attribute :gtins_count, :integer, default: 0
end
```

`gtins_count` is the virtual attribute. I was using at first the `attr_accessor` ruby keyword to defined the getter / setter of the attributes, but gtins_count must be of type integer so I used the [attributes](https://api.rubyonrails.org/classes/ActiveRecord/Attributes/ClassMethods.html) keyword which is introduced in rails 5. It is better because you can specify the attribute type, while using `attr_accessor` in ruby will always make you have a string through the getter. 👎

## Step 3, the controller.
__epc_orders_controller.rb__
{% highlight ruby %}
class Admin::EpcOrdersController < Admin::AdminController
  def create
    @epc_order = EpcOrder.new(epc_params)
    if @epc_order.save
      # success steps
    end
  end

  private

  def epc_params
    params.require(:epc_orders).permit(:description, :gtins_count)
  end  
end
{% endhighlight %}

This is the controller of the resource, the important thing to see here is we add the `gtins_count` variable in the permitted params of the model.

Once the three steps has be done, you can simply use the attributes like if it is part of the model :

## Step 4, the model, again.
__epc_order.rb__
{% highlight ruby %}
class EpcOrder < ApplicationRecord
  after_create :create_gtins

  def create_gtins
    gtins_count.times do
      GtinEpcOrder.create
    end
  end
end
{% endhighlight %}

