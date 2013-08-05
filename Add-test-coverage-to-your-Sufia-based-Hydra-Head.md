After Installing Sufia..

Add rspec-rails and factory-girl to your Gemfile 
```
  group :development, :test do
    gem "rspec-rails"
    gem "factory_girl_rails"
  end
```

Run `bundle install`

Run the rspec generator `rails g rspec:install`

Run `rake spec`

… everything should pass -- 0 tests, 0 failures

Put this test into spec/controllers/  

https://github.com/projecthydra/sufia/blob/master/spec/controllers/dashboard_controller_spec.rb
... Then add this before block after the line that reads `describe DashboardController do`: 

```
  before do
    @routes = Sufia::Engine.routes
  end
```

In order to run this test, you need to:

Add these lines to your spec/spec_helper.rb

```
FactoryGirl.definition_file_paths = [File.expand_path("../factories", __FILE__)]
FactoryGirl.find_definitions

module FactoryGirl
  def self.find_or_create(handle, by=:email)
    tmpl = FactoryGirl.build(handle)
    tmpl.class.send("find_by_#{by}".to_sym, tmpl.send(by)) || FactoryGirl.create(handle)
  end
end
```
Copy this file into spec/factories/
https://github.com/projecthydra/sufia/blob/master/spec/factories/users.rb

Now run `rake spec` again.