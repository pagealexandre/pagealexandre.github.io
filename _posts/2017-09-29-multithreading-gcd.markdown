---
layout: post
title: Swift - Grand Central Dispatch (GCD)
---

Thread is a fondamental notion not only in Swift but in computer science generally, and if you build a serious application at one moment or the other you will need them.

## Threads and Concurrency

Concurrency is basically doing multiple tasks at the same time. Each tasks is executed by a thread. In iOS these threads are managed independently by the OS. We can think of threads as a highway :


![Highway](/assets/highway.jpg)

Each car represent a task and each lane represent a Queue. When you go on the highway in the USA, you can notice sometimes an HOV lane, represented by an diamond on the floor and generally on the left : they are reserved for car with two three people inside, its goal is to reduces pollution and traffic. The main thread on Swift should follow the same idea : we don't want it to be busy because this is where we are going to apply UI task, so the fastest, the best. The background heavy tasks should operate on the right, i.e on the other threads.


## Grand Central Dispatch
To make developer's life easier, Apple has built Grand Central Dispatch (GCD) a low level API built on top of threads. This API allow to easily create and manage threads. The only thing we have to do is giving the API a queue. So now you are probably asking : *What is a queue ?*

We can make an analogy with people waiting in line for an event. The line is the queue and each person is a task. 
There are two type of queue : Serial and Concurrent one.


## Serial Queue
The important notion with Serial Queue is : *First In First Out* (FIFO). 

---Task4---Task3---Task2---Task1--->

Task2 does not start until Task1 is finished. Task3 does not start until Task2 is finished etc... There is a predictable execution Order. And since they are executed one by one they prevent from race condition issue.


## Concurrent Queue
In a concurrent Queue every task start in order, however the order of completion is unpredictable : Task2 can finish before Task1.

-----Task2----------->\\
--Task1-------------->\\
--------Task3-------->\\
-----------Task4----->

A concurrent Queue is faster because things are done concurrently (every task are executed at the same time) however, the completion order is changeable, which can lead to race condition issue. Now that you have the good knowledge, let's dive into the code.


## DispatchQueue
The DispatchQueue object manages the queue. The main and global queues are available : they are for UI task and heavy task such as Fetching JSON Data from a REST API respectively. Let's highlight the difference between a serial queue and a concurrent queue through some code :

{% highlight swift %}
let queue = DispatchQueue.global(qos: .background) // concurrent queue

queue.async {
    for _ in 0..<10 {
        print("🔴")
    }
}

queue.async {
    for _ in 0..<10 {
        print("🔵")
    }
}
{% endhighlight %}

Which result to :

{% highlight text %}
🔴🔵🔵🔴🔵🔴🔵🔴🔵🔴🔵🔴🔵🔴🔵🔴🔵🔴🔵🔴
{% endhighlight %}

It is possible to modify the qos parameter, basically it manages the priority, e.g with two background queue (global) having `.background` and `.userInteractive` we can expect this type of result :

{% highlight text %}
🔵🔴🔵🔴🔵🔴🔵🔴🔵🔴🔵🔵🔵🔵🔴🔵🔴🔴🔴🔴
{% endhighlight %}


Now to demonstrate the principle of a serial queue, let's take the main queue.

{% highlight swift %}
DispatchQueue.main.async {
    for _ in 0..<10 {
        print("🔴")
    }
}

DispatchQueue.main.async {
    for _ in 0..<10 {
        print("🔵")
    }
}
{% endhighlight %}

Result :

{% highlight text %}
🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
{% endhighlight %}

In these two example we use the `async` method to perform the action. Just as a note :
- In a synchronous block, the main threads get the control back once the function is finished.
- In a asynchronous block, the code is executed on a parallel thread but the main thread is not blocked. Thus the asynchronous function does not block the behaviour of an application.

{% highlight swift %}
let queue = DispatchQueue.global(qos: .background) // Concurrent Queue

queue.sync {
    for _ in 0..<10 {
        print("🔴")
    }
}

queue.sync {
    for _ in 0..<10 {
        print("🔵")
    }
}
{% endhighlight %}

Will produce :

{% highlight text %}
🔴🔴🔴🔴🔴🔴🔴🔴🔴🔴🔵🔵🔵🔵🔵🔵🔵🔵🔵🔵
{% endhighlight %}

**Be careful** this does not mean that the thread is not concurrent ! It still is.
When the main thread reach line 3, a new task is added to the background queue. The background thread execute the code to print the red dot. During this time, the main thread is blocked and wait for the synchronous function to return. When the 10 iterations are done, the main thread continue on line 9 and another task is added to the background queue. The background queue execute and once it is finished the main thread get the control back.

In contrast with the snippet of code of the main queue (Serial Queue), the main thread arrive at the first block and a new task is added to the main queue, however the main thread continue to run and the block which print the blue dot is added to the main Queue too. Then the the first block is executed (red dot) and then the second (blue dot).

 *Synchronous function and Asynchronous function in a Serial Queue accomplish basically the same thing however, there is a difference in term of layer where this operate.*

The main queue run on the main threads and is a *serial queue*. The global queue is running by background threads and is concurrent.

## DispatchWorkItem

{% highlight swift %}
let queue = DispatchQueue.global(qos: .background)

let job = DispatchWorkItem {
    for _ in 0..<10 {
        print("🔴")
    }
}

queue.async(execute: job)

{% endhighlight %}

## Final case
It is possible to combine DispatchWorkItem and Notify, I currently did not find any solution to pass parameter to the notify function. 

{% highlight swift %}

var fetch: NSData?
let imageURL = URL(string:"https://upload.wikimedia.org/wikipedia/commons/d/d9/Arduino_ftdi_chip-1.jpg")

let job = DispatchWorkItem {
    self.fetch = NSData(contentsOf: imageURL! as URL)
}

job.notify(queue: DispatchQueue.main) {
    if let imageData = self.fetch {
        self.activityIndicator.stopAnimating()
        self.myImage.image = UIImage(data: imageData as Data)
    }
}

DispatchQueue.global(qos: DispatchQoS.userInteractive.qosClass).async(execute: job)    
{% endhighlight %}

I agree it is the more idiomatic way to express background however semanticaly speaking, It is not clear and intuitive. This leads us to the following snippet of code which although simpler I found easier to read and of course proceed the exact same thing :

{% highlight swift %}

let imageURL = URL(string:"https://upload.wikimedia.org/wikipedia/commons/d/d9/Arduino_ftdi_chip-1.jpg")

DispatchQueue.global(qos: .background).async {
	let fetch = NSData(contentsOf: imageURL! as URL)
	DispatchQueue.main.async {
		if let imageData = fetch {
			self.myImage.image = UIImage(data: imageData as Data)
		}
	}
}

{% endhighlight %}

###What is the plan ?

- Highlight difference between serial and concurrency : done
- QoS class and priority : done
- Async / Sync : done

- DispatchQueue.global appelle le thread ou la Queue ? Je pense la Queue
- WorkItem and notify : done
- CustomQueue



