After Installing Sufia..

Add rspec-rails and factory-girl to your Gemfile and run bundle install
```
  group :development, :test do
    gem "rspec-rails"
    gem "factory_girl_rails"
  end
```
Run the rspec generator   
'rails g rspec'

Run `rake spec`

… everything should pass -- 0 tests, 0 failures

Put this test into spec/controllers/
https://github.com/projecthydra/sufia/blob/master/spec/controllers/dashboard_controller_spec.rb

In order to run this test, you need to:

Add these lines to your spec/spec_helper.rb

```
FactoryGirl.definition_file_paths = [File.expand_path("../factories", __FILE__)]
FactoryGirl.find_definitions
```
Copy this file into spec/factories/
https://github.com/projecthydra/sufia/blob/master/spec/factories/users.rb

Now run `rake spec` again.