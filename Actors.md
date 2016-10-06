When you create or update a Work in curation_concerns, there are a number of processing steps involved, and a specific format for doing so. This format is a [stack (last in, first out)](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)). Each frame does one thing, and the whole stack must be executed in the order it's written in. 

Each processing step, or frame, is broken out into a class called an Actor. They do common tasks like add a Work to a collection, add a Work to another Work, assign a representative file to Work, and apply order to a Work.

Code that wishes to use the whole stack of actors should call `CurationConcerns::CurationConcern.actor.` That method returns an instance of `CurationConcerns::ActorStack`, which is the full list of operations to perform to create or update a Work.

This `ActorStack` is instantiated by an `ActorFactory`.  The default factory is: `CurationConcerns::ActorFactory`, but you can change it by setting `CurationConcerns::CurationConcern.actor_factory=`. For example, Sufia wants to use its own factory, so it sets `CurationConcerns::CurationConcern.actor_factory = Sufia::ActorFactory`

Sufia's `ActorFactory` uses a different set of operations than the CurationConcerns version, but other than that it's the same. `Sufia::ActorFactory` inherits from the version in CurationConcerns, and overrides the `stack_actors` class method like so:

```ruby
module Sufia
  class ActorFactory < CurationConcerns::ActorFactory
    def self.stack_actors(curation_concern)
      [CreateWithFilesActor,
       CurationConcerns::AddToCollectionActor,
       CurationConcerns::AssignRepresentativeActor,
       CurationConcerns::AttachFilesActor,
       CurationConcerns::ApplyOrderActor,
       CurationConcerns::InterpretVisibilityActor,
       model_actor(curation_concern),
       CurationConcerns::AssignIdentifierActor]
    end
  end
end
```

Each of the actors in this chain should inherit from `CurationConcerns::AbstractFactory`, and can override `update()` or `create()` as required. In these methods they must call `next_actor.update(attributes)` or `next_actor.create(attributes)` respectively. Here's an example:

```ruby
module CurationConcerns
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