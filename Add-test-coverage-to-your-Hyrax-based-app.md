# NOTE

This document was copied from the Sufia wiki. It is likely outdated and should be considered deprecated.

# Getting started

After Installing Hyrax...

Add rspec-rails and factory-girl to your Gemfile
```ruby
  group :development, :test do
    gem "rspec-rails"
    gem "capybara"
    gem "factory_girl_rails"
  end
```

Run `bundle install`

Run the rspec generator `rails g rspec:install`

Run `rails spec`

… everything should pass -- 0 tests, 0 failures

# Add FactoryGirl, Capybara and Devise support to your rspec environment

Note: This section is closely based on the [spec_helper.rb](https://github.com/projecthydra-labs/hyrax/blob/master/spec/spec_helper.rb) in hyrax.  You can refer to that file to see how it all fits together.

In order to run this type of test, you need to tell rspec to use FactoryGirl and you need to add a convenient find_or_create method to the FactoryGirl module.

Add these lines to your spec/spec_helper.rb

```ruby
require 'capybara/rspec'
require 'capybara/rails'

module FactoryGirl
  def self.find_or_create(handle, by=:email)
    tmpl = FactoryGirl.build(handle)
    tmpl.class.send("find_by_#{by}".to_sym, tmpl.send(by)) || FactoryGirl.create(handle)
  end
end
```

within the `RSpec.configure do |config|` section, add

```
  config.include Devise::TestHelpers, :type => :controller
```

Copy this file into `spec/factories/`
https://github.com/projecthydra-labs/hyrax/blob/master/spec/factories/users.rb

# Add your first test and make it pass

This is a relatively sophisticated test that attempts to log into your DashboardController and send a couple requests to it.

### Copy the DashboardController test file from Hyrax

Put this test into `spec/controllers/`
https://github.com/projecthydra-labs/hyrax/blob/master/spec/controllers/dashboard_controller_spec.rb

### Clean up the test

Delete the 2 tests that are marked `pending` in `spec/controllers/dashboard_controller.rb` (lines 10-41).  You don't want them.

### Make your test use the routes from Hyrax

The route to dashboard_controller is defined by Hyrax.  By default, rspec tests will only know about the routes defined in your application.  To make the tests in this file aware of the Hyrax routes, add this before block after the line that reads `describe DashboardController do`:

```ruby
  before do
    @routes = Hyrax::Engine.routes
  end
```

### Make sure jetty is configured and running

If you have never done it, you might need to push your solr_config into jetty
```
rails jetty:stop
rails jetty:config
```

Now start jetty (if it's not already running)
```
rails jetty:start
```

### Run the test

Now run `rails spec` again.

If you get a Connection Refused error, it's because jetty is not running.
