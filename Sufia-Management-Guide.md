The Sufia Management Guide provides tips for how to manage, customize, and enhance your Sufia application.

* [Feature matrix](#feature-matrix)
* [Production concerns](#production-concerns)
  * [Identifier state](#identifier-state)
  * [Derivatives](#derivatives)
  * [Web server](#web-server)
  * [Database](#database)
  * [Mailers](#mailers)
* [Background workers](#background-workers)
* [Audiovisual transcoding](#audiovisual-transcoding)
* [User interface](#user-interface)
* [Integration with Dropbox, Box, etc\.](#integration-with-dropbox-box-etc)
* [Autocomplete with geonames](#geonames)
* [Analytics and usage statistics](#analytics-and-usage-statistics)
  * [Capturing usage](#capturing-usage)
  * [Displaying usage in the UI](#displaying-usage-in-the-ui)
* [Zotero integration](#zotero-integration)
* [Customizing metadata](#customizing-metadata)
* [Admin Users](#admin-users)
* [Migrating data to PCDM in Sufia 7](#migrating-data-to-pcdm-in-sufia-7)

## Feature matrix

| Feature | How to enable |
| ------- | ------------- |
| Multiple file upload | automatically enabled |
| Recursive folder upload | automatically enabled (requires Chrome browser)
| Flexible user- and group-based access controls | automatically enabled |
| Generation and validation of identifiers | automatically enabled |
| Generation of derivatives | automatically enabled |
| Fixity checking | automatically enabled |
| Version control | automatically enabled |
| Characterization of uploaded files | automatically enabled |
| Forms for batch editing metadata | automatically enabled |
| Faceted search and browse | automatically enabled |
| Social media interaction | automatically enabled |
| User profiles | automatically enabled |
| User dashboard for file management | automatically enabled |
| Highlighted files on profile | automatically enabled |
| Sharing w/ groups and users | automatically enabled |
| User notifications | automatically enabled |
| Activity streams | automatically enabled |
| Single-use links | automatically enabled |
| Google Scholar-specific metadata embedding | automatically enabled |
| Schema.org microdata, Opengraph/Twitter cards | automatically enabled |
| User-managed collections for grouping files | automatically enabled |
| Full-text indexing & searching | automatically enabled |
| Responsive, fluid, Bootstrap 3-based UI | automatically enabled |
| Featured works and researchers on homepage | automatically enabled |
| Proxy deposit and transfers of ownership | automatically enabled |
| Questioning Authority | automatically enabled |
| ResourceSync capability lists and resource lists | automatically enabled |
| Contact form | automatically enabled |
| Administrative dashboard, w/ feature flippers | automatically enabled (for administrative users) |
| Flexible object model | automatically enabled: to allow zero-file works, enable in Sufia initializer |
| Transcoding of audio and video files | off by default: enable in CurationConcerns initializer (requires `ffmpeg`) |
| Administrative sets (curated collections) | off by default: enable by flipping on in administrative UI |
| Customizable banner image | specify a banner image in Sufia initializer |
| Capture usage statistics | requires [Google Analytics ID to be specified in Sufia initializer](https://github.com/projecthydra/sufia/wiki/Sufia-Management-Guide#capturing-usage) || Geonames integration for location-oriented metadata | requires a Geonames API username to be specified in Sufia initializer |
| Virus detection for uploaded files | install `clamav` package and follow [the instructions in CurationConcerns](https://github.com/projecthydra/curation_concerns#virus-detection) |
| Display usage statistics in the UI | requires [configuration](https://github.com/projecthydra/sufia/wiki/Sufia-Management-Guide#displaying-usage-in-the-ui) |
| Citation formatting suggestions | requires configuration: enable `config.citations` in Sufia initializer, then run `rails g sufia:citation_config` |
| Integration with Zotero | requires [configuration](https://github.com/projecthydra/sufia/wiki/Sufia-Management-Guide#zotero-integration) |
| Integration w/ cloud storage providers | requires [configuration](https://github.com/projecthydra/sufia/wiki/Sufia-Management-Guide#integration-with-dropbox-box-etc) |
| Background jobs | requires configuration: though jobs will automatically run via the default in-memory adapter, we recommend using an `ActiveJob` adapter like `Sidekiq` in production environments |

## Production concerns

In production or production-like (e.g., staging) environments, you may want to make changes to the following areas.

### Identifier state

Sufia uses the ActiveFedora::Noid gem to mint [Noid](https://confluence.ucop.edu/display/Curation/NOID)-style identifiers -- short, opaque identifiers -- for all user-created content (including `GenericWorks`, `FileSets`, and `Collections`). The identifier minter is stateful, meaning that it keeps track of where it is in the sequence of minting identifiers so that the minter can be "replayed," for example in a disaster recovery scenario. (Read more about the [technical details](https://github.com/microservices/noid/blob/master/lib/noid/minter.rb#L2-L35).) The state also means that the minter, once it has minted an identifier, will never mint it again so there's no risk of identifier collisions.

Identifier state is tracked in a file that by default is located in a well-known directory in UNIX-like environments, `/tmp/`, but this may be insufficient in production-like environments where `/tmp/` may be aggressively cleaned out. To prevent the chance of identifier collisions, it is recommended that you find a more suitable filesystem location for your system environment. If you are deploying via Capistrano, that location should **not** be in your application directory, which will change on each deployment. If you have multiple instances of your Sufia application, for instance in load-balanced scenarios, you will want to choose a filesystem location that all instances can access. You may change this by uncommenting and changing the value in this line from `config/initializers/sufia.rb` to a filesystem location other than `/tmp/`:

```ruby
# config.minter_statefile = '/tmp/minter-state'
```

### Derivatives

In Sufia 7, derivatives are served from a directory on the filesystem rather than directly from the Fedora repository. If your production environment includes multiple Rails servers, you will want to make sure that they are using a shared filesystem for derivatives. To set this directory, change the value of `config.derivatives_path` in `config/initializers/curation_concerns.rb`. The default value is `tmp/derivatives/` within your application directory, which will cause unexpected behavior in a multi-server configuration.

### Web server

The web server provided by Rails (whether that's WEBrick, Unicorn, or another) is not built to scale out very far, so you should consider alternatives such as Passenger with Apache httpd or nginx.

### Database

The database provided by default is SQLite, and you may wish to swap in something built more for scale like PostgreSQL or MySQL, both of which have been used in other production Sufia applications.

### Mailers

Sufia uses ActionMailer to send email to users. Some environments may need special configuration to enable your application to send messages. These changes are best made in one of your application's environment files. The configuration options are documented in the [ActionMailer Rails Guide](http://guides.rubyonrails.org/action_mailer_basics.html#action-mailer-configuration).

## Background workers

Sufia processes long-running or particularly slow work in background jobs to speed up the web request/response cycle. Sufia (as of version 7.0.0) no longer packages a default queuing backend for background jobs -- all jobs are expressed as ActiveJob instances, so there is a wide variety of backends that you may use that will work with Sufia's background workers. You may want to read more about [ActiveJob](http://guides.rubyonrails.org/active_job_basics.html).

If you'd like to use Resque in your Sufia app, we've written up a [guide](https://github.com/projecthydra/sufia/wiki/Background-Workers-(Resque-in-Sufia-7)) to help you along.

## Audiovisual transcoding

Sufia includes support for transcoding audio and video files via CurationConcerns. View [CurationConcerns README](https://github.com/projecthydra/curation_concerns/blob/master/README.md#ffmpeg) for installation/configuration help.

## User interface

If you encounter problems with the in-browser content editor -- e.g., the About page, and the three blocks on the homepage -- you should **remove** turbolinks support from `app/assets/javascripts/application.js` if present by deleting the following line:

```javascript
//= require turbolinks
```

## Integration with Dropbox, Box, etc.

Sufia provides built-in support for the [browse-everything](https://github.com/projecthydra/browse-everything) gem, which provides a consolidated file picker experience for selecting files from [DropBox](http://www.dropbox.com),
[Skydrive](https://skydrive.live.com/), [Google Drive](http://drive.google.com),
[Box](http://www.box.com), and a server-side directory share.

To activate browse-everything in your sufia app, run the browse-everything install generator

```
rails g browse_everything:install --skip-assets
```

This will generate a file at _config/browse_everything_providers.yml_. Open that file and enter the API keys for the providers that you want to support in your app.  For more info on configuring browse-everything, go to the [project page](https://github.com/projecthydra/browse-everything) on github.

After running the browse-everything install generator and setting the API keys for the desired providers, an extra tab will appear in your app's Upload page allowing users to pick files from those providers and submit them into your app's repository.

**If your `config/initializers/sufia.rb` was generated with sufia 3.7.2 or earlier**, then you need to add this line to an initializer (probably _config/initializers/sufia.rb _):
```ruby
config.browse_everything = BrowseEverything.config
```

## Geonames

In order for autocomplete to work in the location field, you must have a valid geonames account that can query the service and return possible matches. To setup an account, visit: http://www.geonames.org/login.

After registering and getting an account username, you'll need to enable the free web services by going to http://www.geonames.org/manageaccount and clicking the link to enable. Once that's done, and you've verified you can query the service via their REST Api, add the account username to the `config/initializers/sufia.rb` file under `config.geonames_username`.

## Analytics and usage statistics

Sufia provides support for capturing usage information via Google Analytics and for displaying usage stats in the UI.

### Capturing usage

To enable the Google Analytics javascript snippet, make sure that `config.google_analytics_id` is set in your app within the `config/initializers/sufia.rb` file. A Google Analytics ID typically looks like _UA-99999999-1_.

### Displaying usage in the UI

To display data from Google Analytics in the UI, first head to the Google Developers Console and create a new project:

    https://console.developers.google.com/project

Let's assume for now Google assigns it a project ID of _foo-bar-123_. It may take a few seconds for this to complete (watch the Activities bar near the bottom of the browser).  Once it's complete, enable the Google+ and Google Analytics APIs here (note: this is an example URL -- you'll have to change the project ID to match yours):

    https://console.developers.google.com/project/apps~foo-bar-123/apiui/api

Finally, head to this URL (note: this is an example URL -- you'll have to change the project ID to match yours):

    https://console.developers.google.com/project/apps~foo-bar-537/apiui/credential

And create a new OAuth client ID.  When prompted for the type, use the "Service Account" type.  This will give you the OAuth client ID, a client email address, a private key file, a private key secret/password, which you will need in the next step.

Then run this generator:

```
rails g sufia:usagestats
```

The generator will create a configuration file at _config/analytics.yml_.  Edit that file to reflect the information that the Google Developer Console gave you earlier, namely you'll need to provide it:

* The path to the private key
* The password/secret for the privatekey
* The OAuth client email
* An application name (you can make this up)
* An application version (you can make this up)

Lastly, you will need to set `config.analytics = true` and `config.analytic_start_date` in _config/initializers/sufia.rb_ and ensure that the OAuth client email
has the proper access within your Google Analyics account.  To do so, go to the _Admin_ tab for your Google Analytics account.
Click on _User Management_, in the _Account_ column, and add "Read & Analyze" permissions for the OAuth client email address.

## Zotero integration

Integration with Zotero-managed publications is possible using [Arkivo](https://github.com/inukshuk/arkivo). Arkivo is a Node-based Zotero subscription service that monitors Zotero for changes and will feed those changes to your Sufia-based app. [Read more about this work](https://www.zotero.org/blog/feeds-and-institutional-repositories-coming-to-zotero/).

To enable Zotero integration, first [register an OAuth client with Zotero](https://www.zotero.org/oauth/apps), then [install and start Arkivo-Sufia](https://github.com/inukshuk/arkivo-sufia) and then generate the Arkivo API in your Sufia-based application:

```
rails g sufia:arkivo_api
```

The generator does the following:

* Enables the API in the Sufia initializer
* Adds a database migration
* Creates a routing constraint that allows you to control what clients can access the API
* Copies a config file that allows you to specify the host and port Arkivo is running on
* Copies a config file for your Zotero OAuth client credentials

Update your database schema with `rake db:migrate`.

Add unique Arkivo tokens for each of your existing user accounts with `rake sufia:user:tokens`. (New users will have tokens created as part of the account creation process.)

Edit the routing constraint in `config/initializers/arkivo_constraint.rb` so that your Sufia-based app will allow connections from Arkivo. **Make sure this is restrictive as you are allowing access to an API that allows creates, updates and deletes.**

Tweak `config/arkivo.yml` to point at the host and port your instance of Arkivo is running on.

Tweak `config/zotero.yml` to hold your Zotero OAuth client key and secret. Alternatively, if you'd rather not paste these into a file, you may use the environment variables `ZOTERO_CLIENT_KEY` and `ZOTERO_CLIENT_SECRET`.

Restart your app and it should now be able to pull in Zotero-managed publications on behalf of your users.  Each user will need to link their Sufia app account with their Zotero accounts, which can be done in the "Edit Profile" page. After the accounts are linked, Arkivo will create a subscription to that user's Zotero-hosted "My Publications" collection. When users add items to their "My Publications" collection via the Zotero client, they will automatically be pushed into the Sufia-based repository application. Updates to these items will trigger updates to item metadata in your app, and deletes will delete the files from your app.

## Customizing metadata

Chances are you will want to customize the default metadata provided by Sufia.  Here's [a guide](https://github.com/projecthydra/sufia/wiki/Customizing-Metadata) to help you with that in Sufia >= 7.0.0. If you need similar instructions for Sufia 6.x, read more about [customizing metadata in 6.x](https://github.com/projecthydra/sufia/wiki/Customizing-Metadata-(Sufia-6.x)).

## Admin Users

See [making admin users in Sufia](https://github.com/projecthydra/sufia/wiki/Making-Admin-Users-in-Sufia).

## Migrating data to PCDM in Sufia 7

**WARNING: THIS IS IN PROGRESS AND UNTESTED**

1. Create a GenericWork for each GenericFile. The new GenericWork should have the same id as the old GenericFile so that URLs that users have saved will route them to the appropriate location.
1. Create a FileSet for each GenericWork and add it to the `ordered_members` collection on the GenericWork.
1. Move the binary from `GenericFile#content` to `FileSet#original_file`.

Here are more details on a [proof of concept](https://github.com/projecthydra/sufia/wiki/Sufia-6-to-Sufia-7-Migration).

