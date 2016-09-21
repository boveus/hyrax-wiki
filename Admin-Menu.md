# Administration Menu

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
The code for it lives in `app/controllers/concerns/curation_concerns/admin_controller_behavior.rb #index`  You won't 
be changing this method.  It is mentioned here for reference only.

`#index` will render `app/views/curation_concerns/admin/index.html.erb` which will in turn render each of the partials
listed for the action under `actions -> index -> partials`.

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

Click Hello World.  `admin_controller_behavior.rb #index` will render the partial `_hi_there` when you select the `hello_world` menu item
and you will see content `Hi There`.


