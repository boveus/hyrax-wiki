# Setup

## Install hydra-role-management

Follow the directions for installing [hydra-role-management](https://github.com/projecthydra/hydra-role-management#installing).

Add the following to your application's Gemfile:

```ruby
gem 'hydra-role-management'
```

Then install the gem and run its database migrations:

```bash
# each of these commands will produce some output.
bundle install
rails generate roles
rake db:migrate
```

When you add new abilities to `app/models/ability.rb` per the hydra-role-management instructions, add that code to the end of the `custom_permissions` method.

# Add an initial admin user via command-line

Run rails console

```
$ rails c
```

Add the administrative role to an administrative user:

```
admin = Role.create(name: "admin")
admin.users << User.find_by_user_key( "your_admin_users_email@fake.email.org" )
admin.save
```

# Add more admin users

You can add more administrative users via the command-line like you did above, or you can do so via the UI:

* Login as an admin user
* Browse to http://your.app.host/roles 
* Select a role and add one or more users to it

# Confirm user was made an admin

Run the following commands in the rails console:

```
u = User.find_by_user_key( "your_admin_users_email@fake.email.org" )
u.admin?
# => true
```

If `u.admin?` returns `true` then everything worked as expected.

Or you can verify this in the UI:

* Login as an admin user
* Browse to http://your.app.host/roles 
  * The page should load without exceptions
  * You should see a button labeled "Create a new role"

If you don't see this or get a permission error, you may need to restart your Rails server and try again in the browser.
