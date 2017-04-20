The Sufia Development Guide is for people who want to modify Sufia itself. See the [[Sufia Management Guide]] for guidance on how to configure and set up a Sufia-based application.

* [Grab an issue](#grab-an-issue)
* [Setup development environment](#setup-development-environment)
  * [Development prerequisites](#development-prerequisites)
  * [Generate test app](#generate-test-app)
  * [Work with test app in the browser](#work-with-test-app-in-the-browser)
  * [Cleaning up](#cleaning-up)
* [Run the test suite](#run-the-test-suite)
  * [Test prerequisites](#test-prerequisites)
  * [Run the wrappers](#run-the-wrappers)
  * [Run tests](#run-tests)
  * [Troubleshooting / Testing FAQ](#troubleshooting--testing-faq)
* [Start servers individually for development](#start-servers-individually-for-development)
  * [Solr](#solr)
  * [Fedora](#fedora)
  * [Rails](#rails)
* [Install problems](#install-problems)
* [Change validation behavior](#change-validation-behavior)
* [Quick Start for Hyrax development](#quick-start-for-hyrax-development)
* [Regenerating the README TOC](#regenerating-the-readme-toc)

# Grab an issue

If you're interested in picking up an issue in Sufia, feel free to look over the issues marked "ready" in GitHub (or browse the "Ready" column in [Sufia's waffle board](https://waffle.io/projecthydra/sufia)). When you find an issue you'd like to work on, please do assign yourself the issue in GitHub. This is an important step that signals to other developers that you're working on the issue and that they shouldn't pick it up too.

# Setup development environment

Since Sufia is a Rails engine, in order to develop/test new Sufia UI features you'll want to test/demo Rails application (based on Sufia). Luckily, Sufia comes with its own test application which can be used for this purpose. This section walks you through the basics of setting up that test application for development (and testing).

## Development prerequisites

First off, you need to have Sufia installed (obviously):
* Make sure all of Sufia's [basic prerequisites](https://github.com/projecthydra/sufia#prerequisites) are running.
* Additional prerequisite for tests: [PhantomJS](http://phantomjs.org/).
* Git clone the Sufia repo, use latest stable Ruby (2.3.1), run `bundle install`.

## Generate test app

The test application (`<sufia directory>/.internal_test_app`) can be used both for UI development, as well as for [running the test suite](#run-the-test-suite).

*NOTE: Run this only once.*
```
cd <sufia directory>
rake engine_cart:generate
```

This generates the test Sufia application in the `<sufia directory>/.internal_test_app` directory. You should not have to regenerate the test app unless you pull in code changes from the `master` branch, or start working on a new feature or bug.

## Work with test app in the browser

In order to do UI development, you'll want to load the Sufia test app (`<sufia directory>/.internal_test_app`) in your browser.

This section assumes that you have generated the test app via `rake engine_cart:generate`.

1. Only if you are using ActiveFedora earlier than 9.13, you need to copy some configuration files. Check your ActiveFedora version with `cd .internal_test_app; bundle show active-fedora`. If it's 9.13 or higher, you can skip these steps, go on to step 2.

    1. Verify that ActiveFedora has installed the development templates by looking for
  `.internal_test_app/config/solr_wrapper_test.yml`. (Note: ActiveFedora was first released with this change in version
  [9.13.10](https://github.com/projecthydra/active_fedora/releases/tag/v9.13.0).  If the file exists skip to step 3.)

    1. Copy the templates from ActiveFedora

        If you don't have the files you will need to copy them each time you regenerate the test application. This step is a bit hacky and the fixed was added to ActiveFedora in [c8309ae](https://github.com/projecthydra/active_fedora/commit/c8309aecd4672d719271cd98c103f017f25191a1).

      1. Get the following dev environment-related files from ActiveFedora and put them in `.internal_test_app/`:
        * [.fcrepo_wrapper](https://github.com/projecthydra/active_fedora/blob/master/lib/generators/active_fedora/config/fedora/templates/.fcrepo_wrapper)
        * [.solr_wrapper](https://github.com/projecthydra/active_fedora/blob/master/lib/generators/active_fedora/config/solr/templates/.solr_wrapper)

         **Note:** These two files are dot files and are not visible unless you add the `-a` flag to `ls`.

      1. Get the following test environment-related files from ActiveFedora and put them in `.internal_test_app/config`:
         * [fcrepo_wrapper_test.yml](https://github.com/projecthydra/active_fedora/blob/master/lib/generators/active_fedora/config/fedora/templates/fcrepo_wrapper_test.yml)
         * [solr_wrapper_test.yml](https://github.com/projecthydra/active_fedora/blob/master/lib/generators/active_fedora/config/solr/templates/solr_wrapper_test.yml)

1. Run SolrWrapper in development mode. SolrWrapper picks up configuration from the `.solr_wrapper` file. By default ActiveFedora installs a configuration file (to `.internal_test_app/.solr_wrapper`) that starts Solr on port 8983.
  1. Open a terminal
  1. `cd <sufia directory>\.internal_test_app`
  1. `solr_wrapper`

1. Run FcrepoWrapper in development mode. FcrepoWrapper picks up configuration from the `.fcrepo_wrapper` file. By default ActiveFedora installs a configuration file (to `.internal_test_app/.fcrepo_wrapper`) that starts Fedora on port 8984.
  1. Open a terminal
  1. `cd <sufia directory>\.internal_test_app`
  1. `fcrepo_wrapper`

1. Run the Rails server in development mode
  1. Open a terminal
  1. `cd <sufia directory>\.internal_test_app`
  1. `rails server`

1. View the app by opening [localhost:3000](http://localhost:3000) in a web browser.

1. Optionally, if you want to use Sufia's Administrative functionality, you'll need to [make admin users in Sufia](https://github.com/projecthydra/sufia/wiki/Making-Admin-Users-in-Sufia) from within your test application directory (`<sufia directory>\.internal_test_app`)

1. You can now begin develop new features in Sufia (by modifying/adding code in your `<sufia directory>`), and test them out immediately in your web browser. In many cases any changes you make will take immediate effect. But, on occasion, you may find you'll need to restart the Rails server (see step 5 above).

## Cleaning up

1. To stop the development Fedora and Solr servers, press CTRL-C in the terminal windows in which they are running
1. To clean out the data in Solr & Fedora
  1. `cd <sufia directory>\.internal_test_app`
  1. `fcrepo_wrapper clean`
  1. `solr_wrapper clean`

# Run the test suite

## Test prerequisites

Before running the test suite, you need to be sure you've initialized your development environment using these instructions:
* [Development Prerequisites](#development-prerequisites) - Install all of the base development prerequisites
* [Generate test app](#generate-test-app) - Ensure you've generated the Sufia test application in `<sufia directory>/.internal_test_app`

## Run the wrappers
*Note: DO NOT USE FOR PRODUCTION*.
Note: You'll need separate terminal windows/tabs for each wrapper.

### Wrapper Method 1

From `<sufia directory>/.internal_test_app`, if you have `config/solr_wrapper_test.yml` and `config/fcrepo_wrapper_test.yml` (see [Work with test app in the browser](#work-with-test-app-in-the-browser) for more info), run:

```bash
solr_wrapper -v --config config/solr_wrapper_test.yml
fcrepo_wrapper -v --config config/fcrepo_wrapper_test.yml # separate window/tab
```
### Wrapper Method 2

From `<sufia directory>/` (not `.internal_test_app`):

```bash
solr_wrapper -v -d solr/config/ -n hydra-test -p 8985
fcrepo_wrapper -v -p 8986 --no-jms # separate window/tab
```

## Run tests
Run entire suite:
```
cd <sufia directory>
rake spec
```

Run a single spec:
```
rspec path/to/filel_spec.rb
```

Run jasmine server:
```
rake jasmine
```
Access the jasmine server at port 8888. Note rspec's jasmine spec won't run any jasmine file with syntax errors. It does report the the number of specs run; pay attention to that number if you're doing js tests. Insert `debug` into your test file to do browser debugging on the test itself. If you're working on a remote box, add
```
rack_options:
  Host: 0.0.0.0
```
in `spec/javascripts/support/jasmine.yml`

Run Rubocop style checker:
```
rubocop
```

If Rubocop finds style violations, you can ask it to try automatically fixing them. We recommend committing all work prior to running this command, though, as sometimes Rubocop will create breaking changes:
```
rubocop -a
```

## Troubleshooting / Testing FAQ

### The generated test app isn't doing what I expected after making (and/or pulling) changes to Sufia.  What can I do?

Generally, engine_cart will pick up changes to Sufia.  If not, try the following to regenerate a clean test app:

```bash
cd <sufia directory>
rm -rf .internal_test_app Gemfile.lock
bundle install
rake engine_cart:generate
```

### Where is `hydra-jetty`/its rake tasks?

It was retired.  Solr and Fedora now run individually; see [Run the wrappers](#run-the-wrappers).

### How do I verify that Solr is running?

In a web browser, check [localhost:8985](http://localhost:8985/). You should see an instance of Solr with a Solr core name of `hydra-test`

### How do I verify that that Fedora is running?

In a web browser, check [localhost:8986](http://localhost:8986/). You should see the Fedora splash page.

### Hey, those ports (8985/8986) look different from what I expected!

Only because they are! Now that we use `solr_wrapper` and `fcrepo_wrapper` instead of `hydra-jetty`, which bundled test and dev environments together and was occasionally problematic, test and dev instances of Solr and Fedora now run on separate ports. If you want to run the test suite, use the ports above (8985 for Solr and 8986 for Fedora). If you want to check out Sufia in your browser, use port 8983 for Solr and port 8984 for Fedora as stated in [Solr](https://github.com/projecthydra/sufia#start-solr) and [Fedora](https://github.com/projecthydra/sufia#start-fcrepo).

### How do I run the test coverage report?

Just let Travis-CI handle this when you submit your PR. But if you really want to run it locally:

```
COVERAGE=true rspec
```

### Can't you simplify this?

Yes. You can run everything (including the Fedora and Solr wrappers) using the default rake task, like so:

```
rake
```

(Note that the default task in Sufia is the `ci` task, so running the above command is the same as running `rake ci`.)

But note that if you're actively working on a feature or a bug fix, you will likely not want to use this task repeatedly because it's remarkably slower than `rspec`.

### I've pushed my branch, which passed locally, to GitHub and the build failed on Travis-CI. What gives? I need to be able to reproduce this locally to get my branch merged.

The three most common situations here are:

* **Your test application is stale.** Regenerate it.
* **Travis-CI picked up a newer version of a dependency than you have.** Delete your local `Gemfile.lock`, run `bundle install` again, and regenerate the test application.
* **A new version of bundler came out.** Upgrade the version of bundler via `gem update bundler` and then run `bundle install` and regenerate the test application.
* **A new version of Rubygems came out.** Upgrade it via `gem update --system` and then run `bundle install` and regenerate the test application.

# Start servers individually for development

## Solr

If you already have an instance of Solr that you would like to use, you may skip this step.  Open a new terminal window and type:
```
solr_wrapper -d solr/config/ --collection_name hydra-development
```

You can check to see if Solr is started by going to [localhost:8983](http://localhost:8983/).

## Fedora

If you already have an instance of Fedora that you would like to use, you may skip this step.  Open a new terminal window and type:

```
fcrepo_wrapper -p 8984
```

You can check to see if Fedora is started by going to [localhost:8984](http://localhost:8984/).

## Rails

To test-drive your new Sufia application, spin up the web server that Rails provides:

```
rails server
```

# Install problems

If installing Sufia results in errors that look like `ERROR -- : fsevent: running worker failed: Resource temporarily unavailable`, you may include `--skip-listen` among the arguments to `rails new`. That should get you past these errors. NOTE: we do not recommend passing this argument when generating your Sufia application unless you understand what the ramifications of this are.

# Change validation behavior

To change what happens to files that fail validation add an after_validation hook:
```ruby
after_validation :dump_infected_files

def dump_infected_files
  if Array(errors.get(:content)).any? { |msg| msg =~ /A virus was found/ }
    content.content = errors.get(:content)
    save
  end
end
```

# Quick Start for Hyrax development

The following steps need to be done in order to create a test app for Hyrax development.  This is a quick list with links to the details.

1. [Install prerequisite software](#development-prerequisites) - Follow all instructions carefully.
1. Clone [Hyrax](https://github.com/projecthydra-labs/hyrax) code from github
1. Remove existing test app with `rake engine_cart:clean` (Not required after initial clone. Use when your code updates require the test app to be regenerated.)
1. [Create the test app](#generate-test-app) with `rake engine_cart:generate`
1. If using rubyracer for JavaScript runtime, uncomment in `.internal_test_app/Gemfile` and bundle install.  (Not needed if using nodejs.) ([more info](https://github.com/projecthydra-labs/hyrax#javascript-runtime))
1. [Start servers](#start-servers-individually-for-development) with `rake hydra:server`  (e.g. solr, fedora, rails) - Stop with Ctrl-C
1. [Start background workers](https://github.com/projecthydra-labs/hyrax#start-background-workers) (message queue) - several options for message queue
1. Move into the test app directory with `cd .internal_test_app`
1. [Create default administrative set](https://github.com/projecthydra-labs/hyrax#create-default-administrative-set) with `rake hyrax:default_admin_set:create`
1. [Load workflows](https://github.com/projecthydra-labs/hyrax#load-workflows) with `rake hyrax:workflow:load`
1. [Generate a work type](https://github.com/projecthydra-labs/hyrax#generate-a-work-type) with `rails generate hyrax:work Work` (Replace Work with the name of your work type.)

# Regenerating the README TOC

[Install the gh-md-toc tool](https://github.com/ekalinin/github-markdown-toc/blob/master/README.md#installation), then ensure your README changes are up on GitHub, and then run:

`gh-md-toc https://github.com/USERNAME/sufia/blob/BRANCH/README.md`

That will print to stdout the new TOC, which you can copy into `README.md`, commit, and push.
