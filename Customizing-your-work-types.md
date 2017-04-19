After you've generated a new work type (or "curation concern"), you may want to customize it in your application. This is a guide that should help you do this. (If it is missing a section that you would find helpful, please feel free to add it!)

# Making sure routes match your expectations

Rails makes generally good guesses about how to pluralize models, such as the new work type you just generated. Sometimes it gets things wrong, though. Maybe you generate a model called `Atlas`, for example, and Rails guesses that is the plural form of "Atla." If it does, it will assume routing with paths like `atla_path`. If this is something you care about, check `rake routes` after generating the new work type. If it doesn't look the way you'd like, you can add code to your application's `config/initializers/inflections.rb` file like so:

```ruby
ActiveSupport::Inflector.inflections(:en) do |inflect|
  inflect.irregular 'atlas', 'atlases'
end
```

See the [Rails documentation](http://api.rubyonrails.org/classes/ActiveSupport/Inflector/Inflections.html) for more.

# Changing the label and description

When you have two or more work types created, you will notice that the "Create work" and "Batch create" functions pop up a modal that allows you to select a work type. The modal includes a label and description for each work type. To customize the label and description, look for `config/locales/your_work_type.en.yml` in your application and change the following keys `en.hyrax.select_type.{your_work_type).name` and `en.hyrax.select_type.{your_work_type).description`. Restart your application and it should pick up the changes.