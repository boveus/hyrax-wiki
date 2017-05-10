Note: This is intended primarily as a means to share notes prior to the Spring 2017 Hyrax Bootcamp.  Consult the README.md for the official steps.

Tested on Ruby 2.3.1 and OS X 10.12.3

## Dependencies:

1. Imagemagick:
    - `brew install imagemagick --with-openjpeg`
    - run `convert -list format` in terminal and ensure that there is a JP2 entry, imagemagick has been successfully installed with JPEG-2000 support
1. LibreOffice and Ghostscript:
    - install LibreOffice `brew cask install libreoffice`
    - install ghost script `brew install ghostscript`
1. FITS
    - install fits `brew install fits`, note that homebrew currently installs 0.8.6 (which works currently), but Hydra Derivatives is known to be good with up to 1.0.5.  To install a newer FITS follow [these instructions.](https://github.com/projecthydra-labs/hyrax#characterization)
    - ensure fits is installed, `fits -v` should show the version number
1. Redis: 
    - install: `brew install redis`
    - run: `redis-server` 
    - alternatively: `brew services start redis` to launch redis on boot 

## Installing Hyrax

1. clone hyrax: `git@github.com:projecthydra-labs/hyrax.git`
1. enter the folder: `cd hyrax`
1. install gems: `bundle install`
1. generate the test app: `rails engine_cart:generate`
1. enter the test app: `cd .internal_test_app/`
1. start solr: `solr_wrapper`
    - ensure that `http://127.0.0.1:8983/solr/#/hydra-development` resolves in your browser
1. start fedora: `fcrepo_wrapper`
    - ensure that `http://127.0.0.1:8984/rest` resolves in your browser

## Configuring Hyrax

1. ensure you are in the `.internal_test_app` dir
1. configure a workflow: `rails hyrax:workflow:load`
1. configure an admin_set: `rails hyrax:default_admin_set:create`
1. configure fits
    - if you installed via homebrew: 
      1. open `config/initializers/hyrax.rb` and find `config.fits_path` 
      1. uncomment that line and change it to read `config.fits_path = "fits"`
    - if you installed by some other method and `fits.sh` works in terminal, no action is needed
    - if you installed by some other method and neither `fits` nor `fits.sh` works in terminal, you need to edit `config.fits_path` (located in `config/initializers/hyrax.rb`) to point to `fits.sh` in your installation.  You may also need to make `fits.sh` executable.  
1.  configure background jobs:
    - open `config/application.rb`
    - beneath the line `class Application < Rails::Application` add `config.active_job.queue_adapter = :inline`
1.  (optional): if you want to be an admin user
    - open `config/role_map.yml`
    - edit your `development:` block so that it looks like:
    ```
    development:
      archivist:
        - archivist1@example.com
      admin:
        - archivist1@example.com
    ```

## Starting the Application
1. if needed start solr, fedora, and redis:
   - solr: `solr_wrapper`
   - fedora: `fcrepo_wrapper`
   - redis: `redis_server`
1. launch the application:
   - rails web server: `rails s`
   - rails console: `rails c`
1. create a new user:
   - with the server running visit `http://localhost:3000/users/sign_up?locale=en`
   - create a user whose email address is `archivist1@example.com` and use any password you want
   - you will then be able to log in and self deposit word, you will also have admin rights if you granted them to archivist1 in the optional step above
   
  
## Developing

Go up a level to the `hyrax` directory and make changes as you would when doing normal rails development, changes should be reflected in your app.  Of course you make need to restart the server for some changes to be picked up.