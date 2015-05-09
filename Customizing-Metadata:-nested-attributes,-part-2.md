# Overview

This builds on the Customizing Metadata: nested attributes, part 1 but instead creates each nested attribute as a complete ActiveFedora:::Base object instead of using blank nodes.

You have a complex RDF data model that has includes descriptive metadata from one resource included in another. As example, let's say you use Sufia's GenericFile, but with multiple authors, and each author is also its own RDF resource with a first and last name.

# Customizing GenericFile

First, write a test expressing what you want this to look like. If you haven't already, setup your rspec environment:

    bundle exec rails g rspec:install


``` ruby
require 'rails_helper'

describe GenericFile do

  let(:file) do
    GenericFile.create.tap do |f|
      f.apply_depositor_metadata
      f.save!
    end
  end

  describe "setting the title" do
    before { file.title = ["My title"] }
    subject { file.title}
    it { is_expected.to eql ["My title"] }


  end



end 
```
