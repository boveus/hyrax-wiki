# Pre-Requisites: Install and Run Redis
Sidekiq relies on the [Redis](http://redis.io/) key-value store, so Redis must be installed and running
on your system in order for background workers to pick up jobs.

# Install Sidekiq
First we need to install Sidekiq. To do this, we'll add the gem to our `Gemfile`
```
gem 'sidekiq'
```

Install the gem: `bundle install`

# Enable Sidekiq ActiveJob adapter
ActiveJob support a [series of adapters](http://api.rubyonrails.org/classes/ActiveJob/QueueAdapters.html), the default being `:async`. In order to configure ActiveJob to use the Sidekiq adapter, we need to update `config/application.rb` in our application and insert the following to set the adapter:

```
class Application  < Rails::Application
  # ...
  config.active_job.queue_adapter = :sidekiq
  # ...
end
```

As an alternative, you may set individual environments to use different adapters. For example, to configure your development environment to use the `:inline` adapter and production to use `:sidekiq` (though note that the `:inline` adapter will cause all background jobs to be executed in the foreground, which will greatly slow down your development instance)

*config/environments/development.rb*
```
config.active_job.queue_adapter = :inline
```

*config/environments/production.rb*
```
config.active_job.queue_adapter = :sidekiq
```

# Queue name(s)
Sufia uses a specific queue for handling ingest work. In many of the job classes in `app/jobs` you will see the queue declaration as:

```
class CreateWorkJob < ActiveJob::Base
  queue_as CurationConcerns.config.ingest_queue_name
```
`CurationConcerns.config.ingest_queue_name` will [default](https://github.com/projecthydra/curation_concerns/blob/1be404f895c71292ed2614d26022c36b964a9b3b/lib/curation_concerns/configuration.rb#L139-L144) to `:default` unless otherwise specified. If you want to change the queue name for ingest, you can set the queue name to the value of your choice in `config/initializers/curation_concerns.rb` by uncommenting the following line and setting to your choice:

```
# ActiveJob queue to handle ingest-like jobs
config.ingest_queue_name = :ingest
```

# Configuring Sidekiq
Sidekiq allows you to manage queues and their priorities using a YAML configuration file.

Create one in your app as: `config/sidekiq.yml`

The YAML file supplies a list of queues and their priorities. There are also [advanced options](https://github.com/mperham/sidekiq/wiki/Advanced-Options) that you can explore further to customize for the needs of your application. Your initial file may look like the following:
```
---
:queues:
  - default
  - ingest
```

# Configuring Redis
By default Sidekiq will look for Redis on the localhost system at port `6379`. This will 'just work' in a development context if Redis is already running. However, you will likely want to configure Sidekiq to look for Redis in different locations depending on the environment. To do so, you will need a Redis configuration file, and a Sidekiq configuration file. Sidekiq's [Redis guide](https://github.com/mperham/sidekiq/wiki/Using-Redis) can be referenced for further detail.

* Create `config/redis.yml` -- Populate it with the Redis information for each environment. Here is a sample which assumes `REDIS_HOST` and `REDIS_PORT` will be available to the application environment, and has localhost fallbacks for development and test environments:

  ```
  development:
    host: <%= ENV.fetch('REDIS_HOST', localhost) %>
    port: <%= ENV.fetch('REDIS_PORT', 6379) %>
  test:
    host: <%= ENV.fetch('REDIS_HOST', localhost) %>
    port: <%= ENV.fetch('REDIS_PORT', 6379) %>
  production:
    host: <%= ENV.fetch('REDIS_HOST') %>
    port: <%= ENV.fetch('REDIS_PORT') %>
  ```

* Create `config/initializers/sidekiq.rb` -- Populate with references to your `config/redis.yml` file we just made.

  ```
  config = YAML.load(ERB.new(IO.read(Rails.root + 'config' + 'redis.yml')).result)[Rails.env].with_indifferent_access
  
  redis_conn = { url: "redis://#{config[:host]}:#{config[:port]}/" }

  Sidekiq.configure_server do |s|
    s.redis = redis_conn
  end

  Sidekiq.configure_client do |s|
    s.redis = redis_conn
  end
  ```

# Monitoring Sidekiq

Sidekiq comes with a [built-in web application](https://github.com/mperham/sidekiq/wiki/Monitoring#web-ui) that you can mount to monitor the state of your message queue. To setup this application, first mount the application in `config/routes.rb`:

```
require 'sidekiq/web'
mount Sidekiq::Web => '/sidekiq'
```

**For production applications** you will likely want to secure access to this queue interface. There are various ways to do this depending on the scope of your application. Please see the [authentication](https://github.com/mperham/sidekiq/wiki/Monitoring#authentication) section of the Sidekiq wiki for options. Since Sufia uses Devise, the examples given will be available for you to integrate.

# Starting Sidekiq
The most direct way to start Sidekiq is by opening a separate terminal window or tab, ensuring that you are in your project directory, and starting the service (**note that this may require `bundle exec` depending on how you use Ruby**):

```
sidekiq
```

**For production applications**, there are a few ways to deploy and manage sidekiq. If you are using Capistrano, you can consider the following options:

* [Use Upstart and Capistrano for Ubuntu and Centos 6.x](https://github.com/mperham/sidekiq/wiki/Deploying-to-Ubuntu)
* [capistrano-sidekiq plugin](https://github.com/seuros/capistrano-sidekiq)

# Troubleshooting
Please see the Sidekiq wiki [troubleshooting](https://github.com/mperham/sidekiq/wiki/Problems-and-Troubleshooting) page which covers a wide variety of tips and solutions for issues that might arise. Beyond that, there is the [Sidekiq Google Group](https://groups.google.com/forum/#!forum/sidekiq), the [Hydra Tech Google Group](https://groups.google.com/forum/#!forum/hydra-tech) as well as the [Hydra #dev Slack channel](https://wiki.duraspace.org/pages/viewpage.action?pageId=43910187#Getintouch!-Slack).