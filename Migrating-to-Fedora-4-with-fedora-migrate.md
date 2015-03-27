## Prerequisites
Make sure you have a running version of Fedora 4. In these instructions we assume that Fedora 4 is running at http://127.0.0.1:8983/fedora/rest

Ensure that you've updated your application to Sufia 6.

## Migrating to Fedora 4
Note: for testing, you can use the [migrate](https://github.com/projecthydra/hydra-jetty/tree/migrate) branch of the hydra-jetty application, which contains both Fedora 3.8 and Fedora 4.1.
* Add the fedora-migrate gem to your Gemfile and update: `gem 'fedora-migrate'`
* Create a `config/fedora3.yml` file which should look exactly like your `config/fedora.yml` from your previous Sufia 5 application
``` ruby
development:
  user: fedoraAdmin
  password: fedoraAdmin
  url: http://127.0.0.1:8983/fedora3
```
* Create a migration rake task similar to
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
* Run `rake migrate`
* Examine `report.json` for the results
6. Run `rake fedora:migrate:reset` to erase all the Fedora 4 data and try again
