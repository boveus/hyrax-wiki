# Administration Menu for Developers of Curation Concerns Code

**Audience:  Developers**

**Development Area: commits to Curation Concerns code**

To view the admin menu, click the user name and select Admin Dashboard.

### One time process to add development admin user.  Not for use in production.

The user will need to be an admin user.  During development, you can create an admin by...

Edit `.internal_test_app/config/role_map.yml`

```
development:
  ...
  admin:
    - adminuser@example.com
```

## Adding new admin menu items

### Files effected:

* lib/curation_concerns/configuration.rb
* app/controllers/curation_concerns/admin_controller.rb
* app/controllers/concerns/curation_concerns/admin_controller_behavior.rb
* app/views/curation_concerns/admin/index.html.erb
* config/routes.rb
* config/locales/curation_concerns.en.yml

### Directories effected: 

* app/sources/curation_concerns
* app/views/curation_concerns/admin
* app/views/curation_concerns/admin/widget
* app/assets/javascripts/curation_concerns
* app/helpers/curation_concerns


### Steps to add a new menu item in the left hand menu of the Administration Menu.

#### Understanding the configuration file

Edit lib/curation_concerns/configuration.rb

Example configuration with one menu item.
```
    def dashboard_configuration
      @dashboard_configuration ||= {
        menu: {
          index: {}
        },
        actions: {
          index: {
            partials: [
              "total_example",
              "total_objects",
              "total_objects_pie"
            ]
          }
        },
        data_sources: {
          resource_stats: CurationConcerns::ResourceStatisticsSource
        }
      }
    end
```

Menu items appear in the left hand side by adding a menu and defining its action.  In the above example, there is a 
single left hand menu item defined by `menu -> index` with `action -> index`.

There is a default controller method that will drive the rendering of left hand menu items and their content.  
The code for it lives in `app/controllers/concerns/curation_concerns/admin_controller_behavior.rb #index`  

The `index` method will render `app/views/curation_concerns/admin/index.html.erb` which will in turn render each of the partials
listed for the action under `actions -> index -> partials`.

You won't be changing the `index` method.  It is mentioned here for reference only to understand the processing that is 
happening.

`data_sources` allows you to identify a class to provide data for use in the partials.


#### Add a simple menu item

1) Create a partial to render as the content:  Add file: `app/views/curation_concerns/admin/_hi_there.html.erb`
```
<h1>Hi There</h1>
```

2) Update the config:  

`menu -> hello_world` puts hello_world on the left side admin menu
`actions -> hello_world` tells default `#index` method which partials to show in the content

```
    def dashboard_configuration
      @dashboard_configuration ||= {
        menu: {
          index: {},
          hello_world: {}
        },
        actions: {
          index: {
            partials: [
              "total_example",
              "total_objects",
              "total_objects_pie"
            ]
          },
          hello_world: {
            partials: [
              "hi_there"
            ]
          }
        },
        data_sources: {
          resource_stats: CurationConcerns::ResourceStatisticsSource
        }
      }
    end
```

3) Add new route for action that sends it to the index action for processing.  Update the admin namespace to add hello_world.

```
  namespace :admin do
    get :test
    get :hello_world
  end
```

4) Add language translation for the text that will appear in the left hand admin menu.

Edit file `config/locales/curation_concerns.en.yml` and search for `curation_concerns: dashboard: menu:` and add your menu item

```
en:
  curation_concerns:
    ...
    dashboard:
      menu:
        index:  "Admin Dashboard"
        hello_world: "Hello World"
```

5) Test in the browser at http://localhost:3000/admin

Click Hello World.  `admin_controller_behavior.rb #index` will render the partial `_hi_there` 
and you will see content `Hi There`.


#### Adding special processing in the admin controller

By default, the `admin_controller_behavior.rb #index` method will render the partials.  If you need special 
processing that is relatively simple, you can add a separate method for pre-processing before rendering 
in `admin_controller_behavior.rb`

**Pre-req:**  Follow the instructions for Add a simple menu item.

Parts that are unchanged:

* configuration
* language translation

Parts to add/modify:

1) Modify the content partial to use a variable set in the controller.  Edit file: `app/views/curation_concerns/admin/_hi_there.html.erb`
```
<h1>Hi There <%= @name %></h1>
```

2) Add `#hello_world` method to admin controller and set variable for use in partial.

Edit `app/controllers/concerns/curation_concerns/admin_controller_behavior.rb`

```
def hello_world
  @name = current_user.email
  index
end
```

By calling method `index`, you are allowing that method to render all the partials after the setup.  Alternately, you could 
call the rending yourself.

3) Modify route to send it to the hello_world action.  Remove the one previously in the namespace, so that it matches...

```
  namespace :admin do
    get :test
  end
  get '/admin/hello', to: 'admin#hello_world'
```

NOTE: This defines a route to send `/admin/hello` to the `hello_world` method in the admin controller. 

4) Test in the browser at http://localhost:3000/admin

Click Hello World.  `admin_controller_behavior.rb #hello_world` will render the partial `_hi_there` and you will see 
content `Hi There _your_email_`.


#### Using a data source

Pre-req:  Follow the instructions for Add a menu item with special processing.  

Parts that are unchanged:

* language translation

Parts to add/modify:

1) Create data source to provide the name.  Add file: `app/sources/curation_concerns/hello_source.rb`

```
module CurationConcerns
  class HelloSource

    def user_name
      current_user.email
    end

  end
end
```

2) Update the configuration to add the datasource.  Edit `lib/curation_concerns/configuration.rb` and add `hello_data` 
datasource.

```
data_sources: {
  resource_stats: CurationConcerns::ResourceStatisticsSource,
  hello_data: CurationConcerns::HelloSource
}
```

3) Modify partial to use datasource.  Edit file: `app/views/curation_concerns/admin/_hi_there.html.erb`

```
<h1>Hi There <%= hello_data.user_name %></h1>
```

4) Remove `hello_world` method from admin controller.  Edit file: `app/controllers/concerns/curation_concerns/admin_controller_behavior.rb`

Remove `def hello_world` method.

5) Put route back to simple version

NOTE: You could still have additional processing in admin controller and use a data source, depending on the processing required.

```
  namespace :admin do
    get :test
    get :hello_world
  end
```

