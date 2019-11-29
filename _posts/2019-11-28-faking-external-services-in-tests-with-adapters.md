---
layout: post
title: "Faking External Services in Tests with Adapters"
categories: blog
tags: testing, ruby, rails, web
excerpt: >
  When faking external services in tests, start with something simple. I like
  having a public interface to adapters and having an in-memory adapter for
  tests. Let me show you an example.
---

<div class="message">
  This blog post was originally published in thoughtbot's blog. You can find the
  original at <a
  href="https://thoughtbot.com/blog/faking-external-services-in-tests-with-adapters">thoughtbot.com/blog/faking-external-services-in-tests-with-adapters</a>.
</div>



When faking external services in tests, I like to start with a simple solution
and only progress to a more complex one if I need it. That's why I usually start
with a simple class that acts as the public interface to adapters and make an
in-memory adapter for tests. Let's see a concrete example.

## Faking an SMS client

Suppose we need to use Twilio's SMS service. We don't want to make a request to
Twilio's api every time we send a message in a test, and in some tests, we'd
like to assert that the correct messages are being "sent".

Let's first create an `SmsClient` class that will be the interface for the rest
of our code to send messages.

```ruby
class SmsClient
  cattr_accessor :adapter

  def initialize
    @client = adapter.new
  end

  def send_message(args)
    @client.send_message(args)
  end
end
```

We use `cattr_accessor` (that comes with Rails) to have access to a
`SmsClient.adapter` reader and writer methods. If you're not using Rails, we can
instead add an `attr_accessor` to the eigenclass like this:

```ruby
class SmsClient
  class << self
    attr_accessor :adapter
  end

  def initialize
    @client = self.class.adapter.new
  end

  def send_message(args)
    @client.send_message(args)
  end
end
```

The rest of the code is fairly simple. We instantiate our adapter in the
initializer and delegate calls to the `@client` instance.

Now to our adapters:

```ruby
module SmsAdapters
  class Twilio
    def initialize
      # initialize twilio client with credentials
    end

    def send_message(args)
      # send real message to twilio
    end
  end
end
```

```ruby
module SmsAdapters
  class InMemory
    cattr_accessor :messages
    self.messages = []

    def send_message(message)
      self.class.messages << OpenStruct.new(message)
    end
  end
end
```

We use another `cattr_accessor` in our in-memory adapter for ease of collecting
messages at the class level, but we could have just as easily used an
`attr_accessor :messages` on the eigenclass.

The behavior is much like [ActionMailer::Base.deliveries] &mdash; during tests,
action mailer simply collects all emails in an array instead of sending them. In
a similar way, our in-memory adapter collects an array of messages instead of
sending them.

[ActionMailer::Base.deliveries]: https://api.rubyonrails.org/v5.2.3/classes/ActionMailer/Base.html

We can now set the default adapter to be the Twilio adapter:

```diff
 class SmsClient
   cattr_accessor :adapter
+  self.adapter = SmsAdapters::Twilio

   def initialize
     @client = adapter.new
   end

   def send_message(args)
     @client.send_message(args)
   end
 end
```

And we only swap it with our in-memory adapter in tests:

```ruby
# spec/rails_helper.rb

RSpec.configure do |config|
  config.around do |example|
    # keep old adapter to use after test runs
    old_adapter = SmsClient.adapter

    # set our in-memory adapter
    SmsClient.adapter = SmsAdapters::InMemory

    # run the test
    example.run

    # put back the previous adapter
    SmsClient.adapter = old_adapter
  end
end
```

Don't forget to clear that array of messages before every test run. We don't
want some tests polluting other tests' data and causing intermittent failures.

```ruby
# spec/rails_helper.rb

RSpec.configure do |config|
  config.before do
    SmsAdapters::InMemory.clear_messages
  end
end
```

Let's add that `.clear_messages` method to our `InMemory` adapter.

```diff
 module SmsAdapters
   class InMemory
     cattr_accessor :messages
     self.messages = []
+
+    def self.clear_messages
+      self.messages = []
+    end

     def send_message(message)
       self.class.messages << OpenStruct.new(message)
     end
   end
 end
```

Nicely done! Now, all tests that tangentially interact with the `SmsClient` (but
which do not care about the sent messages) continue running normally, being none
the wiser. And when we want to assert that an SMS message has been "sent" in an
integration test, we can simply query the in-memory adapter:

```ruby
it 'sends an sms when opening new account' do
  user = create(:user, phone: "+15552223333")

  OnboardUser.run(user)

  sent_messages = SmsClient.adapter.messages
  expect(sent_messages.last).to have_attributes(to: user.phone, body: "Welcome!")
end
```

### Bonus: development adapter

If we also decide not to send SMS messages in development, we can easily create
a new adapter that uses local storage &mdash; similar to how [ActiveStorage's
local adapter] just saves attachments to disk.

[ActiveStorage's local adapter]: https://edgeguides.rubyonrails.org/active_storage_overview.html#setup

```ruby
module SmsAdapters
  class Local
    def send_message(message)
      # save_to_file
    end
  end
end
```

We could even expose those messages through an `/sms_previews` route in
development. Nice and easy!

## What next?

If you don't like the in-memory type of adapters, or if you find your setup
requires something more complex, these blog posts are a good guide on setting up
fakes with local servers using Sinatra:

* <https://thoughtbot.com/blog/how-to-stub-external-services-in-tests>
* <https://thoughtbot.com/blog/faking-apis-in-development-and-staging>

Happy testing!
