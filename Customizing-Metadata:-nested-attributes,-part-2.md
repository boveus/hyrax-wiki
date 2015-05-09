# Overview

This builds on the Customizing Metadata: nested attributes, part 1 but instead creates each nested attribute as a complete ActiveFedora:::Base object instead of using blank nodes.

You have a complex RDF data model that has includes descriptive metadata from one resource included in another. As example, let's say you use Sufia's GenericFile, but with multiple authors, and each author is also its own RDF resource with a first and last name.

# Test your GenericFile

First, write a test expressing what you want this to look like. If you haven't already, setup your rspec environment:

    bundle exec rails g rspec:install

Write your test in a file called `spec/models/generic_file_spec.rb`

``` ruby
require 'rails_helper'

describe GenericFile do

  let(:file) do
    GenericFile.create.tap do |f|
      f.apply_depositor_metadata "user"
      f.save!
    end
  end

  describe "setting the title" do
    before { file.title = ["My Favorite Things"] }
    subject { file.title}
    it { is_expected.to eql ["My Favorite Things"] }
  end

  describe "adding an author" do
    before { file.authors_attributes = [{first_name: "John", last_name: "Coltrane"}] }
    subject { file.authors.first }
    it { is_expected.to be_kind_of Author }
  end

end
```

Run the test and the first one should pass, but the second one will fail.

# Build your models

Create `app/models/generic_file.rb` and build your model:

``` ruby
class GenericFile < ActiveFedora::Base
  include Sufia::GenericFile

  has_and_belongs_to_many :authors, predicate: ::RDF::DC.creator, class_name: "Author", inverse_of: :generic_files
  accepts_nested_attributes_for :authors

end
```

Create `app/models/author.rb`

``` ruby
class Author < ActiveFedora::Base

  type ::RDF::FOAF.Person

  has_many :generic_files, inverse_of: :authors, class_name: "GenericFile"

  property :first_name, predicate: ::RDF::FOAF.firstName, multiple: false  do |index|
    index.as :stored_searchable
  end

  property :last_name, predicate: ::RDF::FOAF.lastName, multiple: false  do |index|
    index.as :stored_searchable
  end

end
```

Run your test again and it should pass. At this point you can add additional nested terms and start expanding your model tests to cover additional needs, but you'll eventually have to wire up some controllers that will allow your forms to pass these nested attributes' values on to your models.
