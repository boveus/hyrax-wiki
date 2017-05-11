## 
_This has solution has been tested with Hyrax 1.0.0.rc1_

[Legato](https://github.com/tpitale/legato), the Google Analytics API gem used by Hyrax, depends on the eager loading of models extended by `Legato::Model`. This is a problem in development environments where models are not eagerly loaded.

If you are running into this problem, you would be seeing an error like this when you try to view a work's analytics page:

```
ActionView::Template::Error (undefined method `hyrax__pageview' for #<Legato::Management::Profile:0x007fc8ee884a78>):
    1: <!-- Adapted from jquery-flot examples https://github.com/flot/flot/blob/master/examples/visitors/index.html -->
    2: <%= javascript_tag do %>
    3:   var hyrax_item_stats = <%= @stats.to_flot.to_json.html_safe %>;
    4: <% end %>
    5: 
    6: <%= content_tag :h1, @stats, class: "lower" %>
  
hyrax (1.0.0.rc1) app/models/hyrax/statistic.rb:36:in `ga_statistics'
hyrax (1.0.0.rc1) app/models/hyrax/statistic.rb:51:in `combined_stats'
hyrax (1.0.0.rc1) app/models/hyrax/statistic.rb:24:in `statistics'
hyrax (1.0.0.rc1) app/presenters/hyrax/work_usage.rb:27:in `pageviews'
hyrax (1.0.0.rc1) app/presenters/hyrax/work_usage.rb:20:in `to_flot'
```

In order to test Hyrax's google analytics integration in development, add the following initializer to your application at `config/initializers/legato.rb`:

``` ruby
Rails.application.config.after_initialize do
  # If Google Analytics are enabled, pre-load the models which rely on
  # Legato::Model.extended(base) dynamic profile method generation. Non-production
  # environments wouldn't necessarily have this eager loaded by default.
  if Hyrax.config.analytics
    require 'legato'
    require 'hyrax/pageview'
    require 'hyrax/download'
  end
end
```

This initializer is *not* needed in production environments. 

These instructions were adapted from this ticket: [https://github.com/projecthydra/sufia/issues/2460](https://github.com/projecthydra/sufia/issues/2460)