When you create or update a Work in Hyrax, there are a number of processing steps involved, and a specific format for doing so. This format is a [stack (last in, first out)](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)). Each frame does one thing, and the whole stack must be executed in the order it's written in.

Each processing step, or frame, is broken out into a class called an Actor. They do common tasks like add a Work to a collection, add a Work to another Work, assign a representative file to Work, and apply order to a Work.

Code that wishes to use the whole stack of actors should call `Hyrax::CurationConcern.actor.` That method returns an instance of `Hyrax::ActorStack`, which is the full list of operations to perform to create or update a Work.

This `ActorStack` is instantiated by an `ActorFactory`.  The default factory is: `Hyrax::DefaultMiddlewareStack`, but you can change it by setting `Hyrax::CurationConcern.actor_factory=`.

Each of the actors in this chain should inherit from `Hyrax::AbstractFactory`, and can override `update()` or `create()` as required. In these methods they must call `next_actor.update(attributes)` or `next_actor.create(attributes)` respectively. Here's an example:

```ruby
module Hyrax
  class ApplyOrderActor < AbstractActor
    def update(attributes)
      ordered_member_ids = attributes.delete(:ordered_member_ids)
      apply_order(ordered_member_ids) && next_actor.update(attributes)
    end

    private

      def apply_order(new_order)
        ...
      end
  end
end
```
