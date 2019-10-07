---
layout: post
title: A singleton used for configuration couple with store_accessor
permalink: a-singleton-configuration-model.html
categories: ruby rails
---

{: .intro }
Hey guys today I wanna talk about something I encounter during the project I am working on in my company. We'll see what is a singleton (as a reminder) and how to implement it with an hstore feature.
The advantage of this is having dynamical attributes rather than a migration.

The singleton is a class that can be instantiated only once. This is pretty cool for a logger or configuration for the web. Another use for mobile development is a singleton RESTClient, you can invoke from everywhere and fetch data. Most of the example on the internet are with ruby, this is cool in theory but let's see how to accomplish this in practice with Rails. We will have one model representing the singleton.

__shop_configuration.rb__
```rb
class ShopConfiguration < FftServerModel
  include ShopConfigHash
  validate :must_be_singleton, on: :create

  private

  def fill_with_default
    self.config_hash =
      self.class.default_config.
      merge((config_hash || {}).delete_if { |_key, value| value.nil? })
  end

  def must_be_singleton
    errors.add_to_base :must_be_singleton if ShopConfiguration.any?
  end

  class << self
    def singleton
      record = first || new
      record.fill_with_default
      record.save! if record.new_record? || record.changed?
      record
    end
  end
end
```

Here we have a singleton class. We call it through `ShopConfiguration.singleton`. The validation makes it a singleton : there's only one class configuration in the project. If we try to create two configurations class, an error will throw. Note that we use `add_to_base` to add the error.

__shop_config_hash.rb__
```rb
module ShopConfigHash
  include Shipping

  included do
    store_accessor :config_hash, *default_config.keys
  end
end
```

This is where the magic happen. Basically when you'll do something like `ShopConfiguration.singleton.parcel_preparation_ref_validation_method` it will executes the equivalent of `ShopConfiguration.singleton.config_hash.parcel_preparation_ref_validation_method`. The store_access function is in charge to write the value of our content and read it directly in the attribute passed in parameters, i.e the `config_hash`. If you take a look on how the `store_accessor` function is implemented : it defines a setter / getter method and make these function retrieve / set the value from the attribute set as first parameter, i.e `config_hash`. You should think of `store_accessor` as just a shortcut when you call any method, instead of searching for this method at the instance level like a regular attribute, it will go and search inside the given attribute.

__store.rb__
```rb
  def store_accessor(store_attribute, *keys)
    keys = keys.flatten

    _store_accessors_module.module_eval do
      keys.each do |key|
        define_method("#{key}=") do |value|
          write_store_attribute(store_attribute, key, value)
        end

        define_method(key) do
          read_store_attribute(store_attribute, key)
        end
      end
    end
```

One last thing, if you take a look at the snippet below with the splat operator, passing `*default_config.keys` as parameter is just splitting the array of key into multiple argument. Basically it means this :

```rb
def go(x, y)
end

point = [1, 2]
go(*point)
```

__common.rb__
```rb
module ShopConfigHash::Common
  extend ActiveSupport::Concern

  class_methods do
    attr_reader :default_config

    private

    def key(name, type, default: nil)
      @default_config ||= {}
      @default_config[name] = default

      define_singleton_method(name) do
        singleton.send(name)
      end

      define_method :"#{name}=" do |value|
        clazz = "ActiveModel::Type::#{type.capitalize}".constantize
        super(clazz.new.cast(value))
      end
    end
  end
end
```

This code is reusing the same principle as `store_accessor`. All calls like `ShopConfiguration.my_attribute` will be forwarded to the singleton class method (an instance of ShopConfiguration)
When you do `ShopConfiguration.parcel_preparation_ref_validation_method` it will call `ShopConfiguration.singleton.parcel_preparation_ref_validation_method`. 


If you follow the article, this call is forwarded to the `config_hash` attribute. When you do `ShopConfiguration.parcel_preparation_ref_validation_method` as getter or setter you actually call
`ShopConfiguration.singleton.config_hash.parcel_preparation_ref_validation_method`.

One last useful thing to notice is the cast defined in the setter method. What happen is we call on the fly, the `ActiveModel::Type::Boolean` for example and cast the value. 

__shipping.rb__
```rb
module ShopConfigHash::Shipping
  extend ActiveSupport::Concern
  include ShopConfigHash::Common

  included do
    key :parcel_preparation_ref_validation_method, :string

    key :parcel_preparation_ref_validation_strict, :boolean, default: false

    validates :parcel_preparation_ref_validation_strict,
              inclusion: { in: [ true, false ] }
    validates :parcel_preparation_ref_regex,
              presence: true, if: :parcel_prep_ref_regex_validation_method?    
  end

  private

  def parcel_prep_ref_regex_validation_method?
    parcel_preparation_ref_validation_method == 'Regexp'
  end
end
```

This is the final use of the function where you can hard code your attribute. I found this approach really interesting because it allows to split your attributes per file and defined validation for each of them, no needs to have a migration. It can be really useful when you have a ton of settings you don't want to use.
