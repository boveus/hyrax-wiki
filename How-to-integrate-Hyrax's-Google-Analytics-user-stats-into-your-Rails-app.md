# THIS DOCUMENT IS DEPRECATED (BECAUSE OUTDATED AND UNTESTED ON HYRAX)

See the [Hyrax Management Guide](https://github.com/projecthydra-labs/hyrax/wiki/Hyrax-Management-Guide#analytics-and-usage-statistics) for more information on setting up Google Analytics.

***

**Deprecated**:

== Setup & Configuration

* Run the generators
    $ rails g hyrax:models:usagestats
    $ rails g hyrax:models:cached_stats

* Run the migrations
    $ rails db:migrate

* Edit config/initializers/hyrax.rb

  * Set <code>config.analytics = true</code>
  * Set the <code>config.analytic_start_date</code>
  * Set the <code>config.google_analytics_id</code> to your tracking ID (The tracking ID typically looks like UA-99999999-1)

* Edit <code>config/analytics.yml</code> as described in the {README}[https://github.com/projecthydra-labs/hyrax/blob/master/README.md#analytics]

* Run the rake task to build the cache in the <code>user_stats</code> table, and schedule it as a nightly background job

  * <code>$ rails hyrax:stats:user_stats</code>
  * Schedule a nightly job to run that rake task so that user stats will update once a day

* Update views

  If your Rails app overrides the <code>app/views/dashboard/_index_partials/_stats.html.erb</code> file from Hyrax, then you may want to update it to display the user's combined file views & downloads stats.
