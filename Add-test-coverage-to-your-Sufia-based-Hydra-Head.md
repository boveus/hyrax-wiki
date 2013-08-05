After Installing Sufia..

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

Run `rake spec`

… everything should pass -- 0 tests, 0 failures

# Add FactoryGirl, Capybara and Devise support to your rspec environment

Note: This section is closely based on the [spec_helper.rb](https://github.com/projecthydra/sufia/blob/master/spec/spec_helper.rb) in sufia.  You can refer to that file to see how it all fits together.

In order to run this type of test, you need to tell rspec to use FactoryGirl and you need to add a convenient find_or_create method to the FactoryGirl module.

Add these lines to your spec/spec_helper.rb

```ruby
require 'capybara/rspec'
require 'capybara/rails'

FactoryGirl.definition_file_paths = [File.expand_path("../factories", __FILE__)]
FactoryGirl.find_definitions

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
https://github.com/projecthydra/sufia/blob/master/spec/factories/users.rb

# Run your test

Put this test into `spec/controllers/`   
https://github.com/projecthydra/sufia/blob/master/spec/controllers/dashboard_controller_spec.rb  
... Then add this before block after the line that reads `describe DashboardController do`: 

```ruby
  before do
    @routes = Sufia::Engine.routes
  end
```

Now run `rake spec` again.