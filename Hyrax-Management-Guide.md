The Hyrax Management Guide provides tips for how to manage, customize, and enhance your Hyrax application.

* [Feature matrix](#feature-matrix)
* [Production concerns](#production-concerns)
  * [Identifier state](#identifier-state)
  * [Derivatives](#derivatives)
  * [Web server](#web-server)
  * [Database](#database)
  * [Notifications](#notifications)
  * [Mailers](#mailers)
  * [Background workers](#background-workers)
  * [Fixity checking](#fixity-checking)
  * [Virus checking](#virus-checking)
* [Workflows](#workflows)
* [Audiovisual transcoding](#audiovisual-transcoding)
* [Removing a work type](#removing-a-work-type)
* [User interface](#user-interface)
* [Integration with Dropbox, Box, etc\.](#integration-with-dropbox-box-etc)
* [Autocomplete with geonames](#geonames)
* [Analytics and usage statistics](#analytics-and-usage-statistics)
  * [Capturing usage](#capturing-usage)
  * [Capturing download counts](#capturing-download-counts)
  * [Displaying usage in the UI](#displaying-usage-in-the-ui)
  * [Populating the Analytics DB](#populating-the-analytics-db)
  * [Problems with Analytics](#problems-with-analytics)
* [Zotero integration](#zotero-integration)
* [Customizing metadata](#customizing-metadata)
* [Admin users](#admin-users)

# Feature matrix

[[Feature matrix]] (*moved to its own page*)

# Production concerns

In production or production-like (e.g., staging) environments, you may want to make changes to the following areas.

## Identifier state

Hyrax uses the ActiveFedora::Noid gem to mint [Noid](https://confluence.ucop.edu/display/Curation/NOID)-style identifiers -- short, opaque identifiers -- for all user-created content (including `GenericWorks`, `FileSets`, and `Collections`). The identifier minter is stateful, meaning that it keeps track of where it is in the sequence of minting identifiers so that the minter can be "replayed," for example in a disaster recovery scenario. (Read more about the [technical details](https://github.com/microservices/noid/blob/master/lib/noid/minter.rb#L2-L35).) The state also means that the minter, once it has minted an identifier, will never mint it again so there's no risk of identifier collisions.

Identifier state is tracked by default in a file that is located in a well-known directory in UNIX-like environments, `/tmp/`, but this may be insufficient in production-like environments where `/tmp/` may be aggressively cleaned out. To prevent the chance of identifier collisions, it is recommended that you go one of two routes: specifying a less transient shared filesystem location or using a relational database to store minter state.

### Filesystem-backed minter state

You can continue using the filesystem for minter state if you have a system location that your application can write to that is under its control (read: not `/tmp/`). If you are deploying via Capistrano, that location should **not** be in your application directory, which will change on each deployment. If you run multiple instances of your Hyrax application, for instance in load-balanced scenarios, you will want to choose a filesystem location that all instances can access. You may change this by uncommenting and changing the value in this line from `config/initializers/hyrax.rb` to a filesystem location other than `/tmp/`:

```ruby
# config.minter_statefile = '/tmp/minter-state'
```

### Database-backed minter state

Alternatively, to get around situations where you have multiple application instances attempting to obtain locks on a
single, shared filesystem location, you may use the database-backed minter (new to ActiveFedora::Noid 2.x), which stores minter state information in your application's relational database.

To use it, you'll first need to run the install generator:

```bash
$ rails generate active_fedora:noid:install
```

This will create the necessary database tables and seed the database minter. To start minting identifiers with the new minter, override the AF::Noid configuration in e.g `config/initializers/active_fedora-noid.rb`:

```ruby
require 'active_fedora/noid'

ActiveFedora::Noid.configure do |config|
  config.minter_class = ActiveFedora::Noid::Minter::Db
end
```

Using the database-backed minter can cause problems with your test suite, where it is often sensible to wipe out database rows between tests (which destroys the database-backed minter's state, which renders it unusable). To deal with this and still get the benefits of using the database-backed minter in development and production environments, you'll also want to add the following helper to your `spec/spec_helper.rb`:

```ruby
require 'active_fedora/noid/rspec'

RSpec.configure do |config|
  include ActiveFedora::Noid::RSpec

  config.before(:suite) { disable_production_minter! }
  config.after(:suite)  { enable_production_minter! }
end
```

If you switch to the new database-backed minter and want to include in that minter the state of your current file-backed minter, AF::Noid 2.x provides a new rake task that will copy your minter's state from the filesystem to the database:

```bash
$ rails active_fedora:noid:migrate:file_to_database
```

## Derivatives

In Hyrax, derivatives are served from a directory on the filesystem rather than directly from the Fedora repository. If your production environment includes multiple Rails servers, you will want to make sure that they are using a shared filesystem for derivatives. To set this directory, change the value of `config.derivatives_path` in `config/initializers/hyrax.rb`. The default value is `tmp/derivatives/` within your application directory, which will cause unexpected behavior in a multi-server configuration, or if you deploy with capistrano.

## Web server

The web server provided by Rails (whether that's WEBrick, Unicorn, or another) is not built to scale out very far, so you should consider alternatives such as Passenger with Apache httpd or nginx.

## Database

The database provided by default is SQLite, and you may wish to swap in something built more for scale like PostgreSQL or MySQL, both of which have been used in other production Hyrax applications.

## Notifications

As mentioned in the README, Hyrax (as of version 2.0.0) uses a WebSocket-based user notification system, which uses Rails' ActionCable framework. There are known [issues](https://www.phusionpassenger.com/library/config/apache/action_cable_integration/) ([upstream bug](https://github.com/phusion/passenger/issues/1202)) with the WebSocket implementation in Passenger's Apache integration, so notifications will not work with that configuration. Note that Hyrax is smart enough to detect when it's been deployed via Passenger + Apache, and it will automatically disable realtime notifications for you so that it does not start up in a disabled state.

If user notifications are important to your application, consider serving the application via other methods, whether that be Puma, Passenger + Nginx, or [Passenger Standalone via Apache reverse proxy](https://www.phusionpassenger.com/library/deploy/standalone/reverse_proxy.html). In the meantime, you may wish to disable realtime notifications at deploy time -- to do so, set `config.realtime_notifications = false` in `config/initializers/hyrax.rb`.

ActionCable can work with the Redis and PostgreSQL adapters (though notifications will *not* work with the `async` adapter, which is the default). See more about how to configure these adapters in the [ActionCable documentation](http://guides.rubyonrails.org/action_cable_overview.html#subscription-adapter). To enable user notifications, make sure that you have configured ActionCable to use one of the above adapters in your application's `config/cable.yml`. E.g., to use the Redis adapter in the `production` Rails environment:

``` yaml
production:
  adapter: redis
  url: redis://yourhost.yourdomain.edu:6379
```

## Mailers

Hyrax uses ActionMailer to send email to users. Some environments may need special configuration to enable your application to send messages. These changes are best made in one of your application's environment files. The configuration options are documented in the [ActionMailer Rails Guide](http://guides.rubyonrails.org/action_mailer_basics.html#action-mailer-configuration).

## Background workers

[[Background Workers]] (*moved to its own page*)

## Fixity checking

Hyrax provides a [service](https://github.com/samvera/hyrax/blob/master/app/services/hyrax/repository_fixity_check_service.rb) that iterates over all file sets in your repository and verifies fixities in the background. Hyrax will not run this service for you, so you should use a cronjob (or similar, e.g., the `whenever` gem) to run this on a schedule that fits your needs and your content. The code that'll need to run is:

```ruby
Hyrax::RepositoryFixityCheckService.fixity_check_everything
```
## Virus checking

To turn on virus detection, install `clamav` on your system and add the `clamav` gem to your application's `Gemfile`:

```ruby
gem 'clamav'
```

# Workflows

Read more about [[Defining a Workflow]]

# Audiovisual transcoding

Hyrax includes support for transcoding audio and video files via Hyrax. View [Hyrax README](https://github.com/projecthydra-labs/hyrax#trancoding) for installation/configuration help.

# Removing a work type

If you need to remove a work type that you created, e.g., via `rails generate hyrax:work MyWorkType`, you can do so via:

```bash
rails destroy hyrax:work MyWorkType
```

# User interface

If you encounter problems with the in-browser content editor -- e.g., the About page, and the three blocks on the homepage -- you should **remove** turbolinks support from `app/assets/javascripts/application.js` if present by deleting the following line:

```javascript
//= require turbolinks
```

# Integration with Dropbox, Box, etc.

Hyrax provides built-in support for the [browse-everything](https://github.com/projecthydra/browse-everything) gem, which provides a consolidated file picker experience for selecting files from [DropBox](http://www.dropbox.com),
[Skydrive](https://skydrive.live.com/), [Google Drive](http://drive.google.com),
[Box](http://www.box.com), and a server-side directory share.

To activate browse-everything in your hyrax app, run the browse-everything install generator

```
rails g browse_everything:install --skip-assets
```

This will generate a file at _config/browse_everything_providers.yml_. Open that file and enter the API keys for the providers that you want to support in your app.  For more info on configuring browse-everything, go to the [project page](https://github.com/projecthydra/browse-everything) on github.

After running the browse-everything install generator and setting the API keys for the desired providers, an extra tab will appear in your app's Upload page allowing users to pick files from those providers and submit them into your app's repository.

# Geonames

In order for autocomplete to work in the location field, you must have a valid geonames account that can query the service and return possible matches. To setup an account, visit: http://www.geonames.org/login.

After registering and getting an account username, you'll need to enable the free web services by going to http://www.geonames.org/manageaccount and clicking the link to enable. Once that's done, and you've verified you can query the service via their REST Api, add the account username to the `config/initializers/hyrax.rb` file under `config.geonames_username`.

# Analytics and usage statistics

Hyrax provides support for capturing usage information via Google Analytics and for displaying usage stats in the UI.

## Capturing usage

To enable the Google Analytics javascript snippet, make sure that `config.google_analytics_id` is set in your app within the `config/initializers/hyrax.rb` file. A Google Analytics ID typically looks like _UA-99999999-1_.

## Capturing download counts

The mechanism to capture download events is currently broken. See https://github.com/projecthydra-labs/hyrax/issues/968

## Displaying usage in the UI

To display data from Google Analytics in the UI, first head to the Google Developers Console and create a new project:

    https://console.developers.google.com/project

Let's assume for now Google assigns it a project ID of _foo-bar-123_. It may take a few seconds for this to complete (watch the Activities bar near the bottom of the browser).  Once it's complete, enable the Google+ and Google Analytics APIs here (note: this is an example URL -- you'll have to change the project ID to match yours):

    https://console.developers.google.com/apis/library?project=foo-bar-123

Finally, click the 'credentials' menu item and create a new Service Account Key. This will give you the client ID, a client email address, a private key file, and a private key secret/password, which you will need in the next step.

Edit config/analytics.yml to reflect the information that the Google Developer Console gave you earlier; namely you'll need to provide it:

- The path to the private key
- The password/secret for the privatekey
- The Service Account ID (email)
- An application name (you can make this up)
- An application version (you can make this up)

Lastly, you will need to set `config.analytics = true` and `config.analytic_start_date` in _config/initializers/hyrax.rb_ and ensure that the client email
has the proper access within your Google Analyics account.  To do so, go to the _Admin_ tab for your Google Analytics account.
Click on _User Management_, in the _Account_ column, and add "Read & Analyze" permissions for the OAuth client email address.


## Populating the Analytics DB
TODO: Add more detail

The API access required in the UI integration step, above, enables more than just per-object stats display. We can harvest GA stats for all of our objects into the local database, and use this data to integrate usage reports into the Admin Statistics dashboard. So far this integration into the dashboard has not been done.

To harvest stats for all your objects, you might use a rake task that runs Sufia::UserStatImporter which in turn is called by a cron job.


## Problems with Analytics

Having a problem setting up Analytics. See [[Analytics-workaround-for-non-production-environments]], which documents a workaround for one known issue.

# Zotero integration

Integration with Zotero-managed publications is possible using [Arkivo](https://github.com/inukshuk/arkivo). Arkivo is a Node-based Zotero subscription service that monitors Zotero for changes and will feed those changes to your Hyrax-based app. [Read more about this work](https://www.zotero.org/blog/feeds-and-institutional-repositories-coming-to-zotero/).

To enable Zotero integration, first [register an OAuth client with Zotero](https://www.zotero.org/oauth/apps), then [install and start Arkivo-Hyrax](https://github.com/inukshuk/arkivo-hyrax) and then generate the Arkivo API in your Hyrax-based application:

```
rails g hyrax:arkivo_api
```

The generator does the following:

* Enables the API in the Hyrax initializer
* Adds a database migration
* Creates a routing constraint that allows you to control what clients can access the API
* Copies a config file that allows you to specify the host and port Arkivo is running on
* Copies a config file for your Zotero OAuth client credentials

Update your database schema with `rails db:migrate`.

Add unique Arkivo tokens for each of your existing user accounts with `rails hyrax:user:tokens`. (New users will have tokens created as part of the account creation process.)

Edit the routing constraint in `config/initializers/arkivo_constraint.rb` so that your Hyrax-based app will allow connections from Arkivo. **Make sure this is restrictive as you are allowing access to an API that allows creates, updates and deletes.**

Tweak `config/arkivo.yml` to point at the host and port your instance of Arkivo is running on.

Tweak `config/zotero.yml` to hold your Zotero OAuth client key and secret. Alternatively, if you'd rather not paste these into a file, you may use the environment variables `ZOTERO_CLIENT_KEY` and `ZOTERO_CLIENT_SECRET`.

Restart your app and it should now be able to pull in Zotero-managed publications on behalf of your users.  Each user will need to link their Hyrax app account with their Zotero accounts, which can be done in the "Edit Profile" page. After the accounts are linked, Arkivo will create a subscription to that user's Zotero-hosted "My Publications" collection. When users add items to their "My Publications" collection via the Zotero client, they will automatically be pushed into the Hyrax-based repository application. Updates to these items will trigger updates to item metadata in your app, and deletes will delete the files from your app.

# Customizing metadata

Chances are you will want to customize the default metadata provided by Hyrax.  Here's [a guide](https://github.com/projecthydra-labs/hyrax/wiki/Customizing-Metadata) to help you with that in Hyrax.

# Admin users

See [making admin users in Hyrax](https://github.com/projecthydra-labs/hyrax/wiki/Making-Admin-Users-in-Hyrax).
