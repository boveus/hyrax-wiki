## Prerequisites
Make sure you have a running version of Fedora 4. In these instructions we assume that Fedora 4 is running at http://127.0.0.1:8983/fedora/rest

## Upgrading your app
* Update your Gemfile to point to the new Sufia 
`gem 'sufia', ' ~> 6.0.0'`

* Run `bundle install`

* Update your `config/initializers/resque_config.rb` to use the new `redis_namespace` setting. This setting replaces the old `id_namespace`. 

```Resque.redis.namespace = "#{Sufia.config.redis_namespace}:#{Rails.env}"```

* Update `config/fedora.yml` to include the proper URL for Fedora 4 (don't forget the `/rest` at the end) and add a new setting `base_path` A typical development section would look as follows:

```
development:
  user: fedoraAdmin
  password: fedoraAdmin
  url: http://127.0.0.1:8983/fedora/rest
  base_path: /dev
```

* Update your `app/controllers/catalog_controller.rb` as follows: 

1. Replace line `include Blacklight::Catalog` with `include Hydra::Catalog`
1. Insert line `config.search_builder_class = Hydra::SearchBuilder` right after `configure_blacklight do |config|`

The basic structure of your controller would look like this: 
```
class CatalogController < ApplicationController
  include Hydra::Catalog
  [...]
  configure_blacklight do |config|
    config.search_builder_class = Hydra::SearchBuilder
    [...]
  end
end
```

* Do we need to add `gem 'rsolr', '~> 1.0.6'` to the Gemfile (?)

## What's next
[Insert here link to steps to migrate data from Fedora 3 to Fedora 4]

