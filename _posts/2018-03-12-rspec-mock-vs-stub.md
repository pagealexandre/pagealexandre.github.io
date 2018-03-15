---
layout: post
title: Rspec - Stub vs Mock
permalink: rspec-stub-vs-mock.html
categories: ruby rspec stub mock
---

RSpec is a DSL made in Ruby. We will focus on two major detail of RSpec : Stub & Mock

For not beeing to ruby's specific let's see the difference between both
generally. You can use the stub to override the behavior of certain function to return specific value and spy if the function was called or not.

This [post](https://stackoverflow.com/questions/3459287/whats-the-difference-between-a-mock-stub) of stackoverflow has a good explanation on the flow that each a stub and a mock should follow.

# Stub
For the stub, you basically `initialize -> exercice -> verify`. According to [Martin Fowler's website](https://martinfowler.com/articles/mocksArentStubs.html) stub provide canned answers to calls made during the test.

Now, let's briefly talk about spy.
According to [Martin Fowler's website](https://martinfowler.com/articles/mocksArentStubs.html)
> Spies are stubs that also record some information based on how they were called.

So basically a spy is a stub; at least for how the flow should happen, but you use a stub to override the behavior of a function while a spy is counting the number of time a method has been called. I don't think it's relevant to make a dedicated part for spy, because instead of using `have_received` you simply `expect to eq`.

Let's take some ruby code now :

{% highlight ruby %}

class Human
  def run(n)
    n.times { p "Running..." }
  end
end

RSpec.describe do

  let(:human) { Human.new } # Initialize
  before do
    allow(human).to receive(:run)
    human.run(1) # Exercice
  end

  it { expect(human).to have_received(:run).with(1).once } # Verify
end

{% endhighlight %}

Here we initialize the stub, we act by running `human.run` and finally we assert through the expect keyword. When I said the stub is more permissive above, it mean that if we stub the call to `run` but remove `human.run` and `expect(humain).to have_received(:run)...` the test will still pass.

# Mock

Here is the flow for a mock: `initialize -> set expectations -> exercise -> verify`

{% highlight ruby %}
RSpec.describe do
  let (:human) { Human.new } # Initialize
  before { expect(human).to receive(:run).with(1).once } # Set expectation and will verify (after exercice)

  it { human.send(:run, 3) } # Exercice
end
{% endhighlight %}

In a mock you specify the expectation before you exercice / act.

We set expectation and verify on the same line meaning that if your function is not called the mock will throw an error on RSpec. Wait ? How is that possible ? I found this [**link**](https://stackoverflow.com/questions/21210811/in-rspec-what-is-the-difference-between-a-message-expectation-receive-and-a-t) on stackoverflow which explain that `receive` will process if something happen in the future, whereas `have_received` (used for stub) analyze what has happened in the past.

Using mock aren't natural but truely use the power of RSpec.

You see how less line it takes ? And those two test using stubs and mock are checking the exact same thing.
Now the difference with the stub is that, with a mock the test will fail if we don't call `human.run`. When using `expect` the test needs to be validated, i.e receiving run whereas with a stub, will only override the function and return a specify value (if we specify one) but it did not care if the method was actually call. (Except if we expect to have_received a function)

# When to use one or the other ?
Generally, a stub will be used when you want to override a method which call method that you can't always execute in a test environment, for example, let's say you use an external librairy which make REST call to an API, but need to be authenticate to make these calls, instead of authenticated yourself, you just stub the method and specify a static json as return value. This is way simpler to do this and expect a result of an object which takes this json as input.

The mock can be used to check if the method was called and will throw an error if this is not the case (method not called).


## Conclusion

I spent few days reading on mock, stub, spy, I think it's important to know the concept, but at the end, stub and spy are pretty the same thing, and let's be honest you won't remember the difference during all your software engineer career, because they have the same flow, so not important. However, what's important is the difference of flow between a stub and a mock. In the stub
you `initialize`, `run` and then `expect` some result while in a mock, you `initialize`, `set your expectation` and then you `exercice` and `verify`.
