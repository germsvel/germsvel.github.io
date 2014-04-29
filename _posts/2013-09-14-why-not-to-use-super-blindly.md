---
layout: post
title:  "Why not to use super blindly"
date:   2013-09-14 17:14:14
categories: blog
---

Lately I have read and heard a lot of people talk about using `super` to extend behavior instead of using callbacks in Rails. The complaint is that callbacks are hard to test and often violate the simple responsibility principle. Finally, I have heard it said that "super is more OOPish". I don't believe that to be wholly true. Let me explain why.

For those who do not know what `super` does or what callbacks are, let's look at an example to clarify how they work. Suppose you want to do something right after you save some data to a database. Well, in Rails we can use an `after_save` callback:

{% highlight ruby %}

    class YourClass < ActiveRecord::Base
      after_save :do_something

      def do_something
        # do something after saving
      end
    end

{% endhighlight %}

Using `super`, on the other hand, passes the message up through the superclass chain. This way, we can get the behavior described in the superclass and we can add some behavior of our own:

{% highlight ruby %}

    class YourClass < ActiveRecord::Base
      def save
        super
        # do something after saving
      end
    end

{% endhighlight %}

Now I want to make a quick side point. I am not saying that using `super` is never a good idea. Much wiser and knowledgeable developers recommend it, and I am sure there's a good reason for that and a good time to use it. But as a somewhat new, and definitely junior, Ruby developer it is easy to grab ahold of rules of thumb and use them everywhere. It is easy because rules of thumb give you something concrete, something you can hold on to and apply with confidence. But as experience teaches, there is no silver bullet. And that is why new developers have to take these rules with caution.

Back to the main point. I mentioned above that people think callbacks are really hard to test and that they are "less OOPish" than `super`. I think those two statements are actually related and one the cause of the other. Callbacks are hard to test when the design of your application is not very object-oriented. So it is not the callbacks but the design of your application that is making your application hard to test. If your callback is creating dependencies with other classes, then you are violating the single responsibility principle, and your design is at fault. The callback is merely doing what it is meant to do.

In such a case, I think regardless of whether you are using a callback or `super`, you need to extract that method to a different class. For example,

{% highlight ruby %}

    class Cat < Mammal
      after_save: send_reminder_to_feed

      def send_reminder_to_feed
        # send an email reminder to owner
      end
    end

{% endhighlight %}

as well as,

{% highlight ruby %}

    class Cat < Mammal

      def save
        #send email reminder
        super
      end
    end

{% endhighlight %}

both need to factor out `send_email_reminder` to a different class. It is clearly not the responsibility of Cat to know that.

Ok, so you might say, what about the case where the behavior does not violate the single responsibility principle? Which should we use?

Well, if we are talking about callbacks in Rails ActiveRecord models, I believe the only correct use for them is when their logic refers to the internal state of the object. In particular, `after_save` and `after_create` callbacks should only be used when their logic deals with persistence. That is the responsibility of ActiveRecord models in Rails, so that is what they should dedicate themselves to.

Alright, so if an ActiveRecord callback does not deal with the internal state of the object, it is probably violating the single responsibility principle and should be extracted. If it is dealing with the internal state of the object and persistence logic, then we can use it. But what about when we have our own classical inheritance structure with some other classes? We are likely not dealing with persistence logic. What's wrong with using super then?

Well, let's look at an example. Football is our superclass from which Association Football inherits behavior. In the U.S. this type of Football is more commonly known as soccer. We'll call `super` in our initialize method to get the initialize behavior from Football:

{% highlight ruby %}

    class Football
      attr_reader :ball

      def initialize(args)
          @ball = args[:ball]
      end
    end

    class AssociationFootball < Football
      attr_reader :keeper

      def initialize(args)
          @keeper = args[:keeper]
          super(args)
      end
    end

{% endhighlight %}

What's wrong with the code above is that even though our AssociationFootball class inherits the behavior we desire, our subclass knows not only about itself but also about its parent class. Any knowledge that a class has regarding another class creates dependencies, even if that other class is its parent class.

Our AssociationFootball class not only does it now know *how* to implement the initialization method of Football but it also knows *when* it needs to be implemented. We want to get rid of this coupling. In addition, when we, or some other developers in the future, want to add more subclasses, they must now know they have to call `super` from the subclass.

Say we add an American Football (known simply as football in the U.S.) class and we forget to use `super`. An object from the class below will be instantiated without throwing an error.

{% highlight ruby %}

    class AmericanFootball < Football
      attr_reader :kicker

      def initialize(args)
          @kicker = args[:kicker]
      end
    end

{% endhighlight %}

In fact, we won't get an error until we are using a method that is making use of `ball`, which was defined in the Football superclass.

The better alternative is to send hook messages. In the example below we use 'post_initialize' instead of using 'super'. This still lets the subclasses define what gets implemented, but they no longer have to know *when* it gets implemented or how the superclass implements it.

{% highlight ruby %}

    class Football
      attr_reader :ball

      def initialize(args)
         @ball = args[:ball]
         post_initialize(args)
      end

      def post_initialize(args)
        nil
      end
    end

    class AssociationFootball < Football
      attr_reader :keeper

      def post_initialize(args)
          @keeper = args[:keeper]
      end
    end

    class AmericanFootball < Football
      attr_reader :kicker

      def post_initialize(args)
          @kicker = args[:kicker]
      end
    end

{% endhighlight %}

I believe this to be a much better solution. Our classes are less coupled. The subclasses do not need to know how or when the initialize is implemented. In fact, they do not even need to know who is calling the method. All our subclasses know is that they receive a message for 'post_intialize' with some arguments and that they can implement it.

So although I know that testing callbacks is difficult when they are violating the single responsibility principle, I do not recommend blindly using `super` either. I think we ought to use callbacks when they relate to the object in question and Rails `after_save` and `after_create` callbacks only when dealing with persistence logic. If you use classical inheritance, and the superclass is not ActiveRecord (or some other class of which you are not the owner), send hook messages instead and hide the knowledge of how and when to implement behavior from your subclasses.
