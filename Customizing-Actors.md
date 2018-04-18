When you create, update, or destroy a Work in Hyrax, there are a number of processing steps involved, and a specific format for doing so. This format is a [stack (last in, first out)](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)). Each frame does one thing, and the whole stack must be executed in the order it's written in.

Each processing step, or frame, is broken out into a class called an Actor. They do common tasks like add a Work to a collection, add a Work to another Work, assign a representative file to Work, and apply order to a Work.

Code that wishes to use the whole stack of Hyrax actors should call `Hyrax::CurationConcern.actor.` That method (by default) returns an instance of `Hyrax::DefaultMiddlewareStack`, which is the full list of operations to perform in order to create, update, or destroy a Work.

This stack is instantiated by an actor factory. The default factory, as stated above, is `Hyrax::DefaultMiddlewareStack`; if you want complete control over the stack, you can swap in your own customized factory by setting `Hyrax::CurationConcern.actor_factory=` in the Hyrax initializer (`config/initializers/hyrax.rb`).

You can also make targeted changes to Hyrax's actor stack like so (within a `config/initializers/hyrax.rb`):

```ruby
# Adding a new middleware
Hyrax::CurationConcern.actor_factory.use MyCustomActor

# Inserting a new middleware at a specific position
Hyrax::CurationConcern.actor_factory.insert_after Hyrax::Actors::CreateWithRemoteFilesActor, MyCustomActor

# Removing a middleware
Hyrax::CurationConcern.actor_factory.delete Hyrax::Actors::CreateWithRemoteFilesActor

# Replace one middleware with another
Hyrax::CurationConcern.actor_factory.swap Hyrax::Actors::CreateWithRemoteFilesActor, MyCustomActor
```

Each of the actors in this chain should inherit from `Hyrax::AbstractActor`, and can override `#update`, `#create`, and/or `#destroy` as necessary. In these methods, they must call `next_actor.update(env)`, `next_actor.create(env)`, and/or `next_actor.destroy(env)` respectively. Here's an example:

```ruby
module MyApp
  class ReverseOrderActor < AbstractActor
    def update(env)
      ids = env.attributes.delete(:domain_resource_ids)
      reverse_order(ids) && next_actor.update(env)
    end

    private

      def reverse_order(ids)
        ids.reverse
      end
  end
end
```
