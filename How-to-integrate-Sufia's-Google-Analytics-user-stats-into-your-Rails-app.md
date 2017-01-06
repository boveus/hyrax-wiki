**THIS DOCUMENT IS DEPRECATED/OUTDATED/UNTESTED**

See the [Sufia Management Guide](https://github.com/projecthydra/sufia/wiki/Sufia-Management-Guide#analytics-and-usage-statistics) for more information on setting up Google Analytics.

***

== Setup & Configuration

* Run the generators
    $ rails g sufia:models:usagestats
    $ rails g sufia:models:cached_stats

* Run the migrations
    $ rake db:migrate

* Clear (or update) your existing cache

  This step is only necessary if you already have the <code>file_view_stats</code> and <code>file_download_stats</code> tables in your database (from the Sufia 4.2.0 release).

  The new migration added by the cached_stats generator will add a user_id column to the file_view_stats and file_download_stats tables.  If you have existing data in those tables, those rows will be missing the user_id, which means that data won't be available for calculating the total number of file views and downloads for a user.

  There are 2 ways you can correct this problem:

  \1. Delete all the rows in the file_view_stats and file_download_stats tables, and then let the cache rebuild naturally.

  or:

  \2. Write a script that updates all the rows in the file_view_stats and file_download_stats tables to add the user_ids to the existing cache.

* Edit config/initializers/sufia.rb

  * Set <code>config.analytics = true</code>
  * Set the <code>config.analytic_start_date</code> (Note: this was not necessary in Sufia 4.2.0, but it is needed going forward)
  * Set the <code>config.google_analytics_id</code> to your tracking ID (The tracking ID typically looks like UA-99999999-1)

* Edit <code>config/analytics.yml</code> as described in the {README}[https://github.com/projecthydra/sufia/blob/master/README.md#analytics]

* Run the rake task to build the cache in the <code>user_stats</code> table, and schedule it as a nightly background job

  * <code>$ rake sufia:stats:user_stats</code>
  * Schedule a nightly job to run that rake task so that user stats will update once a day

* Update views

  If your Rails app overrides the <code>app/views/dashboard/_index_partials/_stats.html.erb</code> file from Sufia, then you may want to update it to display the user's combined file views & downloads stats.
