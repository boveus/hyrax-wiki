1. Run the server 

  ```
  cd .internal_test_app
  rails s
  ```

1. register a user from a browser window load localhost:3000

  ```
  click login
  click signup
  fill in form and click register
  ```

1. Close your rails server
1. Add your user to the admin role by editing .internal_test_app/config/role_map.yml 

  ```
    development:
      ...
      admin:
        - my_fake_user@faker.com
  ```

1. Load the workflows

  ```
  rake hyrax:workflow:load    
  ```
1. Start your rails server and reload the page (you should now see an administrative menu)
