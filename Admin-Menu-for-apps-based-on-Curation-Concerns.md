# Administration Menu for Apps using Curation Concerns Engine

**Audience:  Developers**
**Development Area:  Apps using Curation Concerns engine**

By default, to view the admin menu, click the user name and select Admin Dashboard.  This comes from the curation concerns
UI.  Your app may have modified this.

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

2) Create/modify configuration:  File: `config/initializers/curation_concerns.rb`

You will either need to copy the entire configuration from Curation Concerns `lib/curation_concerns/configuration.rb` or
update the components of the hash created in Curation Concerns to get the menu items defined there.  Below is an example of 
updating the hash.

`menu -> hello_world` puts hello_world on the left side admin menu
`actions -> hello_world` tells default `#index` method which partials to show in the content

```
  config.dashboard_configuration[:menu][:hello_world] = {}
  config.dashboard_configuration[:actions][:hello_world] = { partials: ['hi_there'] }
  # config.dashboard_configuration[:data_sources][:user_data] = { MyApp::UserDataSource }  # This example doesn't use a datasource
```

3) Add new route for action that sends it to the index action for processing.  Add each menu route in the admin namespace.

```
  CurationConcerns::Engine.routes.draw do
    namespace :admin do
      get :hello_world
    end
  end
```

4) Add language translation for the text that will appear in the left hand admin menu.

Edit file `config/locales/curation_concerns.en.yml`

```
en:
  curation_concerns:
    dashboard:
      menu:
        hello_world:  Hello World
```

5) Create `app/controllers/curation_concerns/admin_controller.rb`

```
module CurationConcerns
  class AdminController < ApplicationController
    include CurationConcerns::AdminControllerBehavior
  end
end
```

Optional:  Remove custom search builders.  As an example, Sufia has customized search builder which had to be removed 
from the admin dashboard processing.  The following demonstrates the code to remove it that is placed in the AdminController class.

```
    def search_builder
      super.except(:add_advanced_parse_q_to_solr)
    end
```

6) Test in the browser at http://localhost:3000/admin

Click Hello World.  `admin_controller_behavior.rb #index` in Curation Concerns engine will render the partial `_hi_there` 
and you will see content `Hi There`.


