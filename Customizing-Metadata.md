# Note

Please note that this documentation applies to Sufia 7.

Table of Contents
=================

* [Override the GenericWork model](#override-the-genericwork-model)
  * [Add the property](#add-the-property)
* [Create a vocabulary](#create-a-vocabulary)
* [Create a service to load the vocabulary](#create-a-service-to-load-the-vocabulary)
* [Create a view to display your field](#create-a-view-to-display-your-field)
* [Update the form so that your new field will display](#update-the-form-so-that-your-new-field-will-display)
  * [If you'd like to make your new field required](#if-you-would-like-to-make-your-new-field-required)
* [Making a default Sufia field non-repeatable](#making-a-default-sufia-field-non-repeatable)
* [Make the field searchable](#make-the-field-searchable)
* [Adjustments to display the field in work display](#adjustments-to-display-the-field-in-work-display)
* [Creating a Default Deposit Agreement](#creating-a-default-deposit-agreement)

# Generate a model

We'll begin by generating a work-type for Sufia to use.  We'll call our work type `GenericWork` although you could use any name you choose (e.g. `Image`, `ETD`, etc).  The first step is to run this generator:

```
$ bin/rails generate sufia:work GenericWork
```

This will generate a number of files into your application.  Now we can customize these files.

# Change the displayed values of the metadata

One file that is generated is the English locale file for the work type.  For `GenericWork` this is located in `config/locales/generic_work.en.yml`.  You can copy this file, change the `en` key to another locale, and translate the included keys to provide internationalization. You may edit the values in the English locale to change what is displayed by default.

# Override the GenericWork model

The GenericWork class is generated with some default metadata, but we want to update it with our own metadata.

```ruby
# app/models/generic_work.rb
class GenericWork < ActiveFedora::Base
  include ::CurationConcerns::WorkBehavior
  include ::CurationConcerns::BasicMetadata
  include Sufia::WorkBehavior
  self.human_readable_type = 'Work'
end
```

## Add the property

We'll add a select box of university departments, and make it a non-repeatable field. Add the property declaration into the `GenericWork` class (and pass a block to make sure the property is indexed in Solr):

```ruby
  property :department, predicate: ::RDF::URI.new("http://lib.my.edu/departments"), multiple: false do |index|
    index.as :stored_searchable, :facetable
  end
```

If you'd like the field to be repeatable your block should look like this:

```ruby
  property :department, predicate: ::RDF::URI.new("http://lib.my.edu/departments") do |index|
    index.as :stored_searchable, :facetable
  end
```

When you're done the file should look like this:

```ruby
class GenericWork < ActiveFedora::Base
  include ::CurationConcerns::WorkBehavior
  include ::CurationConcerns::BasicMetadata
  include Sufia::WorkBehavior
  self.human_readable_type = 'Work'

  property :department, predicate: ::RDF::URI.new("http://lib.my.edu/departments"), multiple: false do |index|
    index.as :stored_searchable, :facetable
  end
end
```

# Create a vocabulary

```ruby
## config/authorities/departments.yml
terms:
  - id: English
    term: English
  - id: History
    term: History
  - id: Latin
    term: Latin
  - id: Zoology
    term: Zoology
```

# Create a service to load the vocabulary

```ruby
# services/departments_service.rb
module DepartmentsService
  mattr_accessor :authority
  self.authority = Qa::Authorities::Local.subauthority_for('departments')

  def self.select_all_options
    authority.all.map do |element|
      [element[:label], element[:id]]
    end
  end

  def self.label(id)
    authority.find(id).fetch('term')
  end
end
```

# Create a view to display your field

```erb
# records/edit_fields/_department.html.erb
<%= f.input :department, as: :select,
    collection: DepartmentsService.select_all_options,
    include_blank: true,
    item_helper: method(:include_current_value),
    input_html: { class: 'form-control' }
%>
```

# Update the form so that your new field will display

```ruby
# forms/curation_concerns/generic_work_form.rb
module CurationConcerns
  class GenericWorkForm < Sufia::Forms::WorkForm
    self.model_class = ::GenericWork
    self.terms += [:department]
  end
end
```

## If you would like to make your new field required

```ruby
# forms/curation_concerns/generic_work_form.rb
module CurationConcerns
  class GenericWorkForm < Sufia::Forms::WorkForm
    self.model_class = ::GenericWork
    self.terms += [:department]
    # Add following line if you'd like to make your field required
    self.required_fields += [:department]
  end
end
```

# Making a default Sufia field non-repeatable

By default all fields in Sufia are repeatable. If you'd like to change this behavior for a field that Sufia provides out of the box, the title field in this case, you can do the following:

```ruby
# forms/curation_concerns/generic_work_form.rb
module CurationConcerns
  class GenericWorkForm < Sufia::Forms::WorkForm
    self.model_class = ::GenericWork
		
    def self.multiple?(field)
      if field.to_sym == :title
        false
      else
        super
      end
    end
		
    # cast title back to multivalued so it will actually deposit
    def self.model_attributes(_)
      attrs = super
      attrs[:title] = Array(attrs[:title]) if attrs[:title]
      attrs
    end
  end
end
```

# Retrieve the field for search results display
Inside app/controllers/catalog_controller.rb, add the following:
```ruby
config.add_show_field solr_name("department", :stored_searchable), label: "Department"
```

# Adjustments to display the field in work display
Add the field to app/models/solr_document.rb
```ruby
def department
  self[Solrizer.solr_name('department')]
end
```

Add the field to app/presenters/sufia/work_show_presenter.rb
```ruby
delegate :department, to: :solr_document
```

Add the field to app/views/curation_concerns/base/_attribute_rows.html.erb
```ruby
<%= presenter.attribute_to_html(:department) %>
```

# Labels and help text

**TBD**

# Creating a Default Deposit Agreement

By default, Sufia will ask you to accept a deposit agreement each time you upload a file. You can make this implicit by having a passive agreement instead. To do this, change the `app/config/initializers/sufia.rb` to:

``` ruby
  config.active_deposit_agreement_acceptance = false
```

Create custom translations in your `sufia.en.yml` locales file:

``` yml
en:
  sufia:
    passive_consent_to_agreement: "By clicking the Save button, I am agreeing to etc..."
    deposit_agreement: "Institutional Agreement"
```

Lastly, create your own `app/views/static/agreement.html.erb` page with the content of your deposit agreement.