The Sufia Development Guide is for people who want to modify Sufia itself. See the [[Sufia Management Guide]] for guidance on how to configure and set up a Sufia-based application.

* [Run the test suite](#run-the-test-suite)
  * [Prerequisites](#prerequisites)
  * [Generate test app](#generate-test-app)
  * [Run the wrappers](#run-the-wrappers)
  * [Run tests](#run-tests)
  * [Troubleshooting / Testing FAQ](#troubleshooting--testing-faq)
* [Work with test app in the browser](#work-with-test-app-in-the-browser)
  * [Cleaning up](#cleaning-up)
* [Change validation behavior](#change-validation-behavior)
* [Regenerating the README TOC](#regenerating-the-readme-toc)

# Run the test suite

## Prerequisites
* Make sure all [basic prerequisites](https://github.com/projecthydra/sufia#prerequisites) are running.
* Additional prerequisite for tests: [PhantomJS](http://phantomjs.org/).
* Tell EngineCart to use Rails 4.x (this is temporary, while we work on Rails 5 support in Sufia): `export RAILS_VERSION=4.2.7`

## Generate test app

*NOTE: Run this only once.*
```
cd <sufia directory>
rake engine_cart:generate
```

This generates `sufia/.internal_test_app` directory.  The tests will run against this test app. You should not have to regenerate the test app unless you pull in code changes from the `master` branch, or start working on a new feature or bug.

## Run the wrappers
*Note: DO NOT USE FOR PRODUCTION*

Start Solr:
```
#  from <sufia root>/.internal_test_app in a separate terminal window 
#  if the file config/solr_wrapper_test.yml exists (see [Work with test app in the browser](#work-with-test-app-in-the-browser) for more info)
solr_wrapper --config config/solr_wrapper_test.yml
# - or - from sufia root in a separate terminal window
solr_wrapper -d solr/config/ -n hydra-test -p 8985
```
Start Fedora:
```
#  from <sufia root>/.internal_test_app in a separate terminal window 
#  if the file config/fcrepo_wrapper_test.yml exists (see [Work with test app in the browser](#work-with-test-app-in-the-browser) for more info)
fcrepo_wrapper --config config/fcrepo_wrapper_test.yml
# - or - from sufia root in a separate terminal window
fcrepo_wrapper -p 8986 --no-jms
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

Run Rubocop style checker:
```
rubocop
```

If Rubocop finds style violations, you can ask it to try automatically fixing them. We recommend committing all work prior to running this command, though, as sometimes Rubocop will create breaking changes:
```
rubocop -a
```

## Troubleshooting / Testing FAQ

* **The generated test app isn't doing what I expected after making (and/or pulling) changes to Sufia.  What can I do?**  Generally, engine cart will pick up changes to Sufia.  If not, try the following to regenerate a clean test app:

```bash
cd <sufia directory>
rm -rf .internal_test_app Gemfile.lock
bundle install
rake engine_cart:generate
```

* **Where is `hydra-jetty`/its rake tasks?**

It was retired.  Solr and Fedora now run individually; see [Run the wrappers](#run-the-wrappers).

* **How do I verify that Solr is running?**

In a web browser, check [localhost:8985](http://localhost:8985/). You should see an instance of Solr with a Solr core name of `hydra-test`

* **How do I verify that that Fedora is running?**

In a web browser, check [localhost:8986](http://localhost:8986/). You should see the Fedora splash page.

* **Hey, those ports (8985/8986) look different from what I expected!**

Only because they are! Now that we use `solr_wrapper` and `fcrepo_wrapper` instead of `hydra-jetty`, which bundled test and dev environments together and was occasionally problematic, test and dev instances of Solr and Fedora now run on separate ports. If you want to run the test suite, use the ports above (8985 for Solr and 8986 for Fedora). If you want to check out Sufia in your browser, use port 8983 for Solr and port 8984 for Fedora as stated in [Solr](https://github.com/projecthydra/sufia#start-solr) and [Fedora](https://github.com/projecthydra/sufia#start-fcrepo).

* **How do I run the test coverage report?**

Just let Travis-CI handle this when you submit your PR. But if you really want to run it locally:

```
COVERAGE=true rspec
```

* **Can't you simplify this?**

Yes. You can run everything (including the Fedora and Solr wrappers) using the default rake task, like so:

```
rake
```

But note that if you're actively working on a feature or a bug fix, you will likely not want to use this task repeatedly because it's remarkably slower than `rspec`.

# Work with test app in the browser

You may want to see the test application in your browser to verify that your changes look correct.  This section assumes that you have generated the test app via `rake engine_cart:generate`.

1. Verify that ActiveFedora has installed the development templates by looking for `.internal_test_app/config/solr_wrapper_test.yml`. (Note: ActiveFedora was first released with this change in version [9.13.10](https://github.com/projecthydra/active_fedora/releases/tag/v9.13.0).  If the file exists skip to step 3.)
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
  1. `rails s`

1. View the app by opening [localhost:3000](http://localhost:3000) in a web browser.

## Cleaning up

1. To stop the Fedora and Solr servers, press CTRL-C in the terminal windows in which they are running
1. To clean out the data in Solr & Fedora
  1. `cd <sufia directory>\.internal_test_app`
  1. `fcrepo_wrapper clean`
  1. `solr_wrapper clean`

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

# Regenerating the README TOC

[Install the gh-md-toc tool](https://github.com/ekalinin/github-markdown-toc/blob/master/README.md#installation), then ensure your README changes are up on GitHub, and then run:

`gh-md-toc https://github.com/USERNAME/sufia/blob/BRANCH/README.md`

That will print to stdout the new TOC, which you can copy into `README.md`, commit, and push.
