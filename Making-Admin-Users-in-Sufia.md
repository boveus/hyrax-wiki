## One time setup for first admin

Follow the directions for installing [hydra-role-management](https://github.com/projecthydra/hydra-role-management#installing).

NOTE: When adding cancan abilities, put them at the end of the custom_permissions method.

## Adding an admin user

Run rails console
```
$ rails c
```

Run the following commands in the rails console
```
r = Role.create name: "admin"
r.users << User.find_by_user_key( "your_admin_users_email@fake.email.org" )
r.save
```

Confirm user was made an admin
```
u = User.find_by_user_key( "your_admin_users_email@fake.email.org" )
u.admin?
  # shows SELECT statment
 => true
```
if u.admin? == true then SUCCESS

Confirm in browser

* go to your Sufia install
* login as the admin user
* add /roles to the end of the main URL

SUCCESS will look like...

* you don't get an error on the /roles page
* you see a button labeled "Create a new role"

If you don't see this or get a permission error, you may need to restart and try again in the browser.