The Sufia Development Guide is for people who want to modify Sufia itself, not an application that uses Sufia.

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
fcrepo_wrapper -p 8984 --no-jms
```
* **Test that Fedora is running.** It should be running at: [localhost:8986](http://localhost:8986/)
* **Those ports look different.** They are! Now that we use `solr_wrapper` and `fcrepo_wrapper` instead of `hydra-jetty`, which bundled test and dev environments together and was occasionally problematic, test and dev instances of Solr and Fedora now run on separate ports. If you want to run the test suite, use the ports above (8985 for Solr and 8986 for Fedora). If you want to check out Sufia in your browser, use port 8983 for Solr and port 8984 for Fedora as stated in the "Creating a Sufia-based app" section above: [Solr](#start-solr) and [Fedora](#start-fcrepo).
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
