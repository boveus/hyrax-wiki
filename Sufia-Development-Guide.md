The Sufia Development Guide is for people who want to modify Sufia itself. See the [[Sufia Management Guide]] for guidance on how to configure and set up a Sufia-based application.

* [Grab an issue](#grab-an-issue)
* [Run the test suite](#run-the-test-suite)
  * [Prerequisites](#prerequisites)
  * [Generate test app](#generate-test-app)
  * [Run the wrappers](#run-the-wrappers)
  * [Run tests](#run-tests)
  * [Troubleshooting / Testing FAQ](#troubleshooting--testing-faq)
* [Work with test app in the browser](#work-with-test-app-in-the-browser)
  * [Cleaning up](#cleaning-up)
* [Start servers individually for development](#start-servers-individually-for-development)
  * [Solr](#solr)
  * [Fedora](#fedora)
  * [Rails](#rails)
* [Install problems](#install-problems)
* [Change validation behavior](#change-validation-behavior)
* [Regenerating the README TOC](#regenerating-the-readme-toc)

# Grab an issue

If you're interested in picking up an issue in Sufia, feel free to look over the issues marked "ready" in GitHub (or browse the "Ready" column in [Sufia's waffle board](https://waffle.io/projecthydra/sufia)). When you find an issue you'd like to work on, please do assign yourself the issue in GitHub. This is an important step that signals to other developers that you're working on the issue and that they shouldn't pick it up too.

# Run the test suite

## Prerequisites
* Make sure all [basic prerequisites](https://github.com/projecthydra/sufia#prerequisites) are running.
* Additional prerequisite for tests: [PhantomJS](http://phantomjs.org/).
* Git clone the repo, use latest stable Ruby (2.3.1), `bundle install`.

## Generate test app

*NOTE: Run this only once.*
```
cd <sufia directory>
rake engine_cart:generate
```

This generates `sufia/.internal_test_app` directory.  The tests will run against this test app. You should not have to regenerate the test app unless you pull in code changes from the `master` branch, or start working on a new feature or bug.

## Run the wrappers
*Note: DO NOT USE FOR PRODUCTION*.  You'll need separate terminal windows/tabs for each wrapper.

### Wrapper Method 1

From `<sufia root>/.internal_test_app`, if you have `config/solr_wrapper_test.yml` and `config/fcrepo_wrapper_test.yml` (see [Work with test app in the browser](#work-with-test-app-in-the-browser) for more info), run:

```bash
solr_wrapper -v --config config/solr_wrapper_test.yml
fcrepo_wrapper -v --config config/fcrepo_wrapper_test.yml # separate window/tab
```
### Wrapper Method 2

From `<sufia root>/` (not `.internal_test_app`):

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

* **A new version of bundler came out.** Upgrade the version of bundler via `gem install bundler -v x.y.z` and then run `bundle install` and regenerate the test application.
* **Travis-CI picked up a newer version of a dependency than you have.** Delete your local `Gemfile.lock`, run `bundle install` again, and regenerate the test application.
* **Your test application is stale.** Regenerate it.

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

# Regenerating the README TOC

[Install the gh-md-toc tool](https://github.com/ekalinin/github-markdown-toc/blob/master/README.md#installation), then ensure your README changes are up on GitHub, and then run:

`gh-md-toc https://github.com/USERNAME/sufia/blob/BRANCH/README.md`

That will print to stdout the new TOC, which you can copy into `README.md`, commit, and push.
