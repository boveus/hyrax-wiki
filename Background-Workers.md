Hyrax processes long-running or particularly slow work in background jobs to speed up the web request/response cycle.

# Backends

Hyrax does not package a default queuing back-end for background jobs. Hyrax builds its jobs using the [Rails ActiveJob framework](http://guides.rubyonrails.org/active_job_basics.html), so there is a wide variety of back-ends that you may use that will work with Hyrax including [Sidekiq](http://sidekiq.org/), [Resque](https://github.com/resque/resque), and [DelayedJob](https://github.com/collectiveidea/delayed_job). Flexibility and choice come with a cost, though, and there's some work involved with integrating whichever queuing back-end you select.

If you'd like to use Sidekiq or Resque in your Hyrax app, we've written up a couple guides on [using Sidekiq with Hyrax](https://github.com/projecthydra-labs/hyrax/wiki/Using-Sidekiq-with-Hyrax) and [using Resque with Hyrax](https://github.com/projecthydra-labs/hyrax/wiki/Using-Resque-with-Hyrax) to help you along.

**We recommend using Sidekiq, which is better maintained and has fewer quirks.**

# Queue Names

A subset of Hyrax's jobs use a shared queue name that is specified in `Hyrax.config.ingest_queue_name` within the Hyrax initializer (`config/initializers/hyrax.rb` in your application). By default, that value is the ActiveJob default, called `:default`. If you need more granular control of which jobs use what queue names, you can control this on a per-class level by including code such as the following in an initializer:

```ruby
StreamNotificationsJob.queue_as :notifications
EventJob.queue_as :events
ImportUrlJob.queue_as :import
AttachFilesToWorkJob.queue_as :attach
```

Or, to put all of Hyrax's jobs on a single queue:

```ruby
Hyrax::ApplicationJob.queue_as :omnibus
Hyrax.config.ingest_queue_name = :omnibus
```