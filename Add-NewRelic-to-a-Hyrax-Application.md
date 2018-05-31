[NewRelic provides documentation](https://docs.newrelic.com/docs/agents/ruby-agent/installation/install-new-relic-ruby-agent) on how to add their service to a Rails application, and doing so for a Hyrax application is no different. However, one of the features of this service doesn't work properly with the Hyrax Middleware (Actor Stack). 

In the `newrelic.yml` configuration, you can disable the instrumentation that causes problems by adding the following configuration to `common` configuration block:

```
common: &default_settings
  ...
  disable_middleware_instrumentation: true
```