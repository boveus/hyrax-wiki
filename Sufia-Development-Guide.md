The Sufia Development Guide is for people who want to modify Sufia itself. See the [[Sufia Management Guide]] for guidance on how to configure and set up a Sufia-based application.

* [Regenerating the README TOC](#regenerating-the-readme-toc)
* [Run the test suite](#run-the-test-suite)
  * [Prerequisites](#prerequisites-1)
  * [Test app](#test-app)
  * [Run tests](#run-tests)
  * [Testing FAQ](#testing-faq)
* [Change validation behavior](#change-validation-behavior)

# Regenerating the README TOC

[Install the gh-md-toc tool](https://github.com/ekalinin/github-markdown-toc/blob/master/README.md#installation), then ensure your README changes are up on GitHub, and then run:

`gh-md-toc https://github.com/USERNAME/sufia/blob/BRANCH/README.md`

That will print to stdout the new TOC, which you can copy into `README.md`, commit, and push.

# Run the test suite

## Prerequisites
* Make sure all [basic prerequisites](#prerequisites) are running.
* Additional prerequisite for tests: [PhantomJS](http://phantomjs.org/).

## Test app
Generate the test app.  *NOTE: Run this only once.*
```
rake engine_cart:generate
```

This generates `sufia/.internal_test_app` directory.  The tests will run against this test app.

## Run tests
```
rake spec
```

## Testing FAQ
* **Where is rake jetty?**  It was retired.  Solr and Fedora are started on their own.
* **How can I start a test instance of Solr? (DO NOT USE FOR PRODUCTION)**
```
# from sufia root in a separate terminal window
solr_wrapper -d solr/config/ -n hydra-test -p 8985
```
* **Test that Solr is running.** It should be running at [localhost:8985](http://localhost:8985/) with a Solr core name of `hydra-test`
* **How can I start a test instance of Fedora? (DO NOT USE FOR PRODUCTION)**
```
# from sufia root in a separate terminal window
fcrepo_wrapper -p 8986 --no-jms
```
* **Test that Fedora is running.** It should be running at: [localhost:8986](http://localhost:8986/)
* **Those ports look different.** They are! Now that we use `solr_wrapper` and `fcrepo_wrapper` instead of `hydra-jetty`, which bundled test and dev environments together and was occasionally problematic, test and dev instances of Solr and Fedora now run on separate ports. If you want to run the test suite, use the ports above (8985 for Solr and 8986 for Fedora). If you want to check out Sufia in your browser, use port 8983 for Solr and port 8984 for Fedora as stated in  [Creating a Sufia-based app](https://github.com/projecthydra/sufia#creating-a-sufia-based-app): [Solr](https://github.com/projecthydra/sufia#start-solr) and [Fedora](https://github.com/projecthydra/sufia#start-fcrepo).
* **The generated test app isn't doing what I expected after making (and/or pulling) changes to Sufia.  What can I do?**  Generally, engine cart will pick up changes to Sufia.  If not, try the following to regenerate the test app:
```bash
rm -rf .internal_test_app Gemfile.lock
bundle install
rake engine_cart:generate
```

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

## Running the test application in Development mode
You may want to see the test application to verify that your changes look correct.  This section assumes that you have generated the test app via rake engine_cart:generate.

1. Verify that active fedora has installed the development templates by looking for `.internal_test_app/config/solr_wrapper_test.yml`.   As of writing this document Active Fedora with this change has yet to be released.  If the file exists skip to step 3.
1. Copy the templates from Actve Fedora

   This step is a bit hacky and should go away once the latest active fedora has been released with this commit: [c8309ae](https://github.com/projecthydra/active_fedora/commit/c8309aecd4672d719271cd98c103f017f25191a1). Unfortunately this will need to be done each time you regenerate engine_cart. 

  1. You want to get the following files from active fedora (for development) and put the in `.internal_test_app/`:
     * [.fcrepo_wrapper](https://github.com/projecthydra/active_fedora/blob/master/lib/generators/active_fedora/config/fedora/templates/.fcrepo_wrapper)
     * [.solr_wrapper](https://github.com/projecthydra/active_fedora/blob/master/lib/generators/active_fedora/config/solr/templates/.solr_wrapper)
   
     ***Note:*** The above files are dot files and are not visible unless you ls -a

  1. You want to get the following files from active fedora (for test) and put the in `.internal_test_app/config`:
     * [fcrepo_wrapper_test.yml](https://github.com/projecthydra/active_fedora/blob/master/lib/generators/active_fedora/config/fedora/templates/fcrepo_wrapper_test.yml)
     * [solr_wrapper_test.yml](https://github.com/projecthydra/active_fedora/blob/master/lib/generators/active_fedora/config/solr/templates/solr_wrapper_test.yml)

1. Run SolrWrapper in development mode.  SolrWrapper will by default pick up the configuration in the .solr_wrapper file.  By default Active Fedora installs a configuration file that starts solr on port 8983.  You may change this default behavior by modifying .internal_test_app/.solr_wrapper 
  1. Open a terminal
  1. `cd .internal_test_app`
  1. `solr_wrapper`

1. run FcrepoWrapper in development mode. FcrepoWrapper will by default pick up the configuration in the .fcrepo_wrapper file.  By default Active Fedora installs a configuration file that starts solr on port 8984.  You may change this default behavior by modifying .internal_test_app/.fcrepo_wrapper
  1. Open a terminal
  1. `cd .internal_test_app`
  1. `fcrepo_wrapper`

1. run the rails server in development mode.
  1. Open a terminal
  1. `cd .internal_test_app`
  1. `rails s`

1. View the app via [localhost:3000](http://localhost:3000)

### cleaning up development

1. To stop the servers press CRTL-C in the terminal window
1. To clean out the data in solr & fedora
  1. `cd .internal_test_app`
  1. `fcrepo_wrapper clean`
  1. `solr_wrapper clean`

