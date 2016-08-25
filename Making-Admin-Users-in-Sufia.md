## One time setup for first admin

### Install hydra-role-management

Follow the directions for installing [hydra-role-management](https://github.com/projecthydra/hydra-role-management#installing).

NOTE: When adding cancan abilities, put them at the end of the custom_permissions method.

### Add the first admin user

Run rails console
```
$ rails c
```

Run the following commands in the rails console
```
u = User.find_by_user_key( "your_admin_users_email@fake.email.org" )
r = Role.create name: "admin"
r.users << u
r.save
```

## Add more admin users

### via Command Line
Run the following commands in the rails console
```
u = User.find_by_user_key( "your_admin_users_email@fake.email.org" )
r = Role.find_by_name("admin")
r.users << u
r.save
```

### via Browser
* Login as an admin user
* add /roles to the end of the main URL
* select a role and add user

## Confirm user was made an admin

### via Command Line

Run the following commands in the rails console
```
u = User.find_by_user_key( "your_admin_users_email@fake.email.org" )
u.admin?
  # shows SELECT statment
 => true
```
if u.admin? == true then SUCCESS

### via Browser

* go to your Sufia install
* login as the admin user
* add /roles to the end of the main URL

SUCCESS will look like...

* you don't get an error on the /roles page
* you see a button labeled "Create a new role"

If you don't see this or get a permission error, you may need to restart and try again in the browser.
