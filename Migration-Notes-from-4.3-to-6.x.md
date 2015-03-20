Oh boy...this will be fun. Hang in there.

## Prerequisites
Make sure you have a running version of Fedora 4. In these instructions we assume that Fedora 4 is running at http://127.0.0.1:8983/fedora/rest

What Jetty should we suggest?
rake jetty:download
rake jetty:start

What about migrations?

* Pre-requisites
A version of Fedora 4 running 
rake jetty:stop
rake jetty:clean

## Upgrading your Sufia 5.x application to use Sufia 6

* Update your Gemfile to point to the new Sufia 
* Update your Gemfile to point to rails 4.2.1 (Is this needed?)
* Run `bundle install`
* bundle update rails (Is this needed?)

* The old `id_namespace` settings is now called `redis_namespace`. Update your `config/initializers/resque_config.rb` to reflect this change. 

`Resque.redis.namespace = "#{Sufia.config.redis_namespace}:#{Rails.env}"`

* Update `config/fedora.yml` to include the proper URL for Fedora 4 (don't forget the `/rest` at the end) and add a new setting `base_path` A typical development section would look as follows:

`    development:
        user: fedoraAdmin
        password: fedoraAdmin
        url: http://127.0.0.1:8983/fedora/rest
        base_path: /dev
`
