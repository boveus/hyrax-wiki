# Overview

This builds on the [Customizing Metadata: nested attributes, part 1](https://github.com/projecthydra/sufia/wiki/Customizing-Metadata:-nested-attributes,-part-1) but instead creates each nested attribute as a complete ActiveFedora:::Base object instead of using blank nodes.

This example is for implementors who have a complex RDF data model that includes descriptive metadata from one resource referenced in another. As example, let's say you use Sufia's GenericFile, but with multiple authors, and each author is also its own RDF resource with a first and last name. We'll build the models and specs required to test them, as well as the controllers that will pass the attributes from the forms to the model, and the specs required to test them too.

This walkthrough assumes you have a blank Sufia 6.0 installation.

# Test your GenericFile

First, write a test expressing what you want this to look like. If you haven't already, setup your rspec environment:

    bundle exec rails g rspec:install

Write your test in a file called `spec/models/generic_file_spec.rb`

``` ruby
require 'rails_helper'

describe GenericFile do

  let(:file) do
    GenericFile.create do |f|
      f.apply_depositor_metadata "user"
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

# Build your controllers

First, we test, but in order to test controllers, we need to add a few extras to our testing suite. Since our controller needs to know who is doing these actions, we need to stub out a user. Sufia uses the FactoryGirl gem for this. Add it to your gem file and include it with other gems in your `:development` and `:test` blocks.

``` ruby
group :development, :test do
  gem 'factory_girl_rails'
end
```

Update your gems:

    bundle exec install

Next, edit `spec/rails_helper.rb` to include test helpers from the Devise gem as well as setup FactoryGirl. You'll want to add the two config lines below to your existing config block

``` ruby

RSpec.configure do |config|
  config.include Devise::TestHelpers, type: :controller
  config.include FactoryGirl::Syntax::Methods
end
```

Additionally, add these methods to the very end of `spec/rails_helper.rb`

``` ruby
FactoryGirl.define do
  factory :user do
    sequence(:email) { |n| "user#{n}@example.com" }
    password 'password'
  end
end

module FactoryGirl
  def self.find_or_create(handle, by=:email)
    tmpl = FactoryGirl.build(handle)
    tmpl.class.send("find_by_#{by}".to_sym, tmpl.send(by)) || FactoryGirl.create(handle)
  end
end
```

Now, let's write our controller test in `spec/controllers/generic_files_controller_spec.rb`

``` ruby
require 'rails_helper'

describe GenericFilesController do
  routes { Sufia::Engine.routes }
  let(:user) { FactoryGirl.find_or_create(:user) }
  before do
    allow(controller).to receive(:has_access?).and_return(true)
    sign_in user
    allow_any_instance_of(User).to receive(:groups).and_return([])
    allow_any_instance_of(GenericFile).to receive(:characterize)
  end

  describe "update" do
    let(:generic_file) do
      GenericFile.create do |gf|
        gf.apply_depositor_metadata(user)
      end
    end

    context "when adding a title" do
      let(:attributes) { { title: ['My Favorite Things'] } }
      before { post :update, id: generic_file, generic_file: attributes }
      subject do
        generic_file.reload
        generic_file.title.first
      end
      it { is_expected.to eql "My Favorite Things"}
    end

    context "when adding an author" do
      let(:attributes) do
        { 
          title: ['My Favorite Things'], 
          authors_attributes: [{first_name: 'John', last_name: 'Coltrane'}],
          permissions_attributes: [{ type: 'person', name: 'archivist1', access: 'edit'}]
        }
      end

      before { post :update, id: generic_file, generic_file: attributes }
      subject { generic_file.reload }

      it "sets the values using the parameters hash" do
        expect(subject.authors.first.first_name).to eql "John"
        expect(subject.authors.first.last_name).to eql "Coltrane"
      end
    end

  end

end
```

The first test should pass, but the second will fail.

# Build out your forms and presenters

Sufia builds its forms based on a Presenter. For GenericFile, this is the Sufia::GenericFilePresenter. We need to create our own presenter so we can include the Author class as an attribute.

First, create `app/presenters/resource_presenter.rb`

``` ruby
class ResourcePresenter < Sufia::GenericFilePresenter
  self.terms = [:title, :authors]
end
```

Next, create the edit and batch edit forms that will use our presenter:

`app/forms/resource_edit_form.rb`

``` ruby
class ResourceEditForm < ResourcePresenter
  include HydraEditor::Form
  include HydraEditor::Form::Permissions
  include NestedAuthors
  self.required_fields = [:title]
end
```

`app/forms/resource_batch_edit_form.rb`

``` ruby 
class ResourceBatchEditForm < ResourceEditForm
end
```

What's NestedAuthors, you ask? This is where we get our form to allow the `authors_attributes` method to pass from the controller to the model. Create `app/forms/nested_authors.rb` with:

``` ruby
module NestedAuthors
  extend ActiveSupport::Concern

  module ClassMethods

    def build_permitted_params
      permitted = super
      permitted << { authors_attributes: permitted_authors_params }
      permitted
    end

    def permitted_authors_params
      [ :id, :_destroy, :first_name, :last_name ]
    end

  end
  
  def authors_attributes= attributes
    model.authors_attributes= attributes
  end

end
```

We have to create the `authors_attributes=` method on the form so that Rails will build the nested forms. ActionView::Helpers is involved [here](https://github.com/rails/rails/blob/a04c0619617118433db6e01b67d5d082eaaa0189/actionview/lib/action_view/helpers/form_helper.rb#L1890)

Last, but certainly not least, we have to tell our controller to use our new ResourcePresenter and forms.
To do this, create `app/controllers/generic_files_controller.rb` with:

``` ruby
class GenericFilesController < ApplicationController
  include Sufia::Controller
  include Sufia::FilesControllerBehavior

  self.presenter_class = ResourcePresenter
  self.edit_form_class = ResourceEditForm

end
```

Now you should be able to rerun your controller spec and have both tests passing.