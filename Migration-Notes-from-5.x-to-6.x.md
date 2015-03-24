## Prerequisites
Make sure you have a running version of Fedora 4. In these instructions we assume that Fedora 4 is running at http://127.0.0.1:8983/fedora/rest

## Upgrading your app
* Update your Gemfile to point to the new Sufia 
`gem 'sufia', ' ~> 6.0.0'`

* Add `gem 'rsolr', '~> 1.0.6'` to your Gemfile

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

### Changes to CatalogController

Update your `app/controllers/catalog_controller.rb` as follows: 

1. Remove require statements: any blacklight, parslet, parsing_nesting
1. Remove include statements: Hydra::Controller::ControllerBehavior, BlacklightAdvancedSearch::ParseBasicQ
1. Rename `config/solr.yml` to `config/blacklight.yml`
1. Remove any field name prefixes such as `desc_metadata__`
1. Replace line `include Blacklight::Catalog` with `include Hydra::Catalog`
1. Insert line `config.search_builder_class = Sufia::SearchBuilder` right after `configure_blacklight do |config|`
1. Change CatalogController.solr_search_params_logic to CatalogController.search_params_logic
1. Add `:add_advanced_parse_q_to_solr` to CatalogController.search_params_logic

The basic structure of your controller would look like this: 
``` ruby
class CatalogController < ApplicationController
  include Hydra::Catalog
  include Sufia::Catalog
  [...]
  CatalogController.search_params_logic += [:add_access_controls_to_solr_params, :add_advanced_parse_q_to_solr]
  [...]
  configure_blacklight do |config|
    config.search_builder_class = Sufia::SearchBuilder
    [...]
  end
end
```

## Migrating to Fedora 4
1. Add the fedora-migrate gem to your Gemfile and update: `gem 'fedora-migrate'`
2. Create a `config/fedora3.yml` file which should look exactly like your `config/fedora.yml` from your previous Sufia 5 application
``` ruby
development:
  user: fedoraAdmin
  password: fedoraAdmin
  url: http://127.0.0.1:8983/fedora3
```
3. Create a migration rake task similar to
``` ruby
require 'fedora-migrate'

module FedoraMigrate::Hooks
  # Apply depositor metadata from Sufia's properties datastream under Fedora 3
  def before_object_migration
    xml = Nokogiri::XML(source.datastreams["properties"].content)
    target.apply_depositor_metadata xml.xpath("//depositor").text
  end
end

desc "Migrates all objects in a Sufia-based application"
task migrate: :environment do
  migration_options = {convert: "descMetadata", application_creates_versions: true}
  migrator = FedoraMigrate.migrate_repository(namespace: "sufia", options: migration_options )
  migrator.report.save
  Rake::Task["sufia:migrate:proxy_deposits"].invoke
  Rake::Task["sufia:migrate:audit_logs"].invoke
end
```
4. Run `rake migrate`
5. Examine `report.json` for the results
6. Run `rake fedora:migrate:reset` to erase all the Fedora 4 data and try again

