# NOTE

This documentation was copied from the Sufia wiki. It may work for Hyrax, but we have not tested this yet.

Table of Contents
=================
* [Prerequisite: Generating the basic files for a new work type](#prerequisite-generating-the-basic-files-for-a-new-work-type)
  * [Labels and help text](#labels-and-help-text)
* [Understanding the controller](#understanding-the-controller)
* [Customizing the work-type to include a single-value text property](#customizing-the-work-type-to-include-a-single-value-text-property)
  * [Add the new single-value property to the model](#add-the-new-single-value-property-to-the-model)
  * [Add the new single-value property to the new/edit form](#add-the-new-single-value-property-to-the-newedit-form)
  * [Add the new single-value property to the show page](#add-the-new-single-value-property-to-the-show-page)
* [Customizing the work-type to include a multi-value text property](#customizing-the-work-type-to-include-a-multi-value-text-property)
  * [Add the new multi-value property to the model](#add-the-new-multi-value-property-to-the-model)
  * [Add the new multi-value property to the new/edit form](#add-the-new-multi-value-property-to-the-newedit-form)
  * [Add the new multi-value property to the show page](#add-the-new-multi-value-property-to-the-show-page)
* [Customizing the work-type to include a controlled vocabulary property](#customizing-the-work-type-to-include-a-controlled-vocabulary-property)
  * [Add the new controlled-value property to the model](#add-the-new-controlled-value-property-to-the-model)
  * [Add the new controlled-value property to the new/edit form](#add-the-new-controlled-value-property-to-the-newedit-form)
    * [Create a vocabulary](#create-a-vocabulary)
    * [Create a service to load the vocabulary](#create-a-service-to-load-the-vocabulary)
  * [Add the new controlled-value property to the show page](#add-the-new-controlled-value-property-to-the-show-page)
* [Modifying default Hyrax fields](#modifying-default-hyrax-fields)
  * [Remove a default property from the set of required fields](#remove-a-default-property-from-the-set-of-required-fields)
  * [Making a default property non-repeatable](#making-a-default-property-non-repeatable)
* [Creating a Default Deposit Agreement](#creating-a-default-deposit-agreement)
* [Labels and help text](#labels-and-help-text)

---
# Prerequisite: Generating the basic files for a new work type

We'll begin by generating a work-type for Hyrax to use.  We'll call our work type `GenericWork` although you could use any name you choose (e.g. `Image`, `ETD`, etc).  The first step is to run this generator:

```
$ bin/rails generate hyrax:work GenericWork
```

This will generate a number of files into your application.  Now we can customize these files.

## Labels and help text

One of the generated files includes `config/locales/generic_work.en.yml` in which many of the labels used on forms and show pages for the new work type are defined.  You can modify these labels and also add labels for any new properties defined.

Application specific labels are defined in `config/locales/hyrax.en.yml`.

# Understanding the controller

The GenericWorksController class is generated with some default behaviors.  It is located at `app/controllers/curation_concerns/generic_works_controller.rb`

```ruby
# Generated via
#  `rails generate curation_concerns:work GenericWork`

module CurationConcerns
  class GenericWorksController < ApplicationController
    include CurationConcerns::CurationConcernController
    # Adds Hyrax behaviors to the controller.
    include Hyrax::WorksControllerBehavior

    self.curation_concern_type = GenericWork
  end
end
```

The model class name follows the Rails convention of controller name minus 'Controller' (e.g. GenericWork)

The form class, used to control the new/edit form, and the presenter class, used to control the show page, can be identified in the controller class.  The default values for these are:

- form_class = model_name.name + Form (e.g. GenericWorkForm) defined in [CurationConcern's work_form_service.rb](https://github.com/projecthydra/curation_concerns/blob/163d6029703bd5eeb77f9e15a7c21e9d5391f8cf/app/services/curation_concerns/work_form_service.rb) self.form_class method
- show_presenter = Hyrax::WorkShowPresenter defined in [Hyrax's works_controller_behavior.rb](https://github.com/projecthydra/hyrax/blob/master/app/controllers/concerns/hyrax/works_controller_behavior.rb)

These can be overridden in the GenericWorksController class using...

```ruby
    self.form_class = GenericWorkExtForm # UNCOMMON, see note below.
    self.show_presenter = GenericWorkPresenter
```

NOTE:
- It is uncommon to override self.form_class as the form class is already generated (e.g. GenericWorkForm) and can hold your extensions.
- The presenter class is not generated.  There is an example adding this class in section [Add the new single-value property to the show page](#add-the-new-single-value-property-to-the-show-page)
- As usual, you can add code for special processing to the controller.

---
# Customizing the work-type to include a single-value text property

## Add the new single-value property to the model

The GenericWork class is generated with some default metadata, but we want to update it with our own metadata.  The generated version of the file looks like...

```ruby
# app/models/generic_work.rb
class GenericWork < ActiveFedora::Base
  include ::CurationConcerns::WorkBehavior
  include ::CurationConcerns::BasicMetadata
  include Hyrax::WorkBehavior
  self.human_readable_type = 'Generic Work'
  # Change this to restrict which works can be added as a child.
  # self.valid_child_concerns = []
  validates :title, presence: { message: 'Your work must have a title.' }
end
```

### Extend the model to add a new single-value property

To define a property that has a single text value, add the following to the GenericWork model.
```ruby
  property :contact_email, predicate: ::RDF::Vocab::VCARD.hasEmail, multiple: false do |index|
    index.as :stored_searchable
  end
```

### Expected behaviors for this property:
- It will be limited to a single value (set multiple: true  or leave off for multi-value, which is the default behavior)
- If included in the new/edit form, it will have `input type=text`  (There is a bit more configuration under section [Add the new single-value property to the new/edit form](#add-the-new-single-value-property-to-the-newedit-form) to have this included in the form.)
- By setting `index.as :stored_searchable`, values will be added to the solr_doc as contact_email_tesi indicating this field is English text (te), stored (s), indexed (i)
  - See [Solr Schema](https://github.com/projecthydra/hydra-head/wiki/Solr-Schema) documentation for more information on dynamic solr field postfixes.
  - See [Solrizer::DefaultDescriptors](http://www.rubydoc.info/gems/solrizer/3.4.0/Solrizer/DefaultDescriptors) documentation for more information on values for `index.as`

| ![QUESTION](https://cloud.githubusercontent.com/assets/6855473/13064236/f2f04cbe-d41e-11e5-9674-e9a56a6326e6.png) | Are all values described in Solrizer::DefaultDescriptors supported within Hyrax/CC property definitions? |
| ------------- | ---------------- |

### The modified model

The GenericWork at this point looks like:

```ruby
class GenericWork < ActiveFedora::Base
  include ::CurationConcerns::WorkBehavior
  include ::CurationConcerns::BasicMetadata
  include Hyrax::WorkBehavior
  self.human_readable_type = 'Generic Work'
  # Change this to restrict which works can be added as a child.
  # self.valid_child_concerns = []
  validates :title, presence: { message: 'Your work must have a title.' }

  property :contact_email, predicate: ::RDF::Vocab::VCARD.hasEmail, multiple: false do |index|
    index.as :stored_searchable
  end
end
```

## Add the new single-value property to the new/edit form

The inclusion of properties in the new/edit form is controlled by the GenericWorkForm class.  The GenericWorkForm class is generated with the default set of properties (aka terms) to include.  The generated version of the file looks like...

```ruby
# Generated via
#  `rails generate curation_concerns:work GenericWork`
module CurationConcerns
  class GenericWorkForm < Hyrax::Forms::WorkForm
    self.model_class = ::GenericWork
    self.terms += [:resource_type]

  end
end
```

NOTE:
- As generated, model_class is the generated model class
- As generated, terms includes basic work terms defined in [CurationConcerns' work_form.rb](https://github.com/projecthydra/curation_concerns/blob/master/app/forms/curation_concerns/forms/work_form.rb) and [Hyrax's work_form.rb](https://github.com/projecthydra/hyrax/blob/master/app/forms/hyrax/forms/work_form.rb).
- A controller class was also generated and configure form_class to be the one described here, e.g., `self.form_class = Hyrax::Forms::GenericWorkForm`

### Adding the property to the work-type's new/edit form

Now we want to update GenericWorkForm to include our own new property.  Edit `app/forms/curation_concerns/generic_work_form.rb` and modify `self.terms` to include the new property.

```ruby
    self.terms += [:resource_type, :contact_email]
```

Optionally, you can add the property to the set of required fields.
```ruby
    self.required_fields += [:contact_email]
```

Optionally, you can also remove one of the default properties from the set of required fields.
```ruby
    self.required_fields -= [:keyword, :rights]
```

The full class after the changes looks like...
```ruby
# app/forms/curation_concerns/generic_work_form.rb
# Generated via
#  `rails generate curation_concerns:work GenericWork`
module CurationConcerns
  class GenericWorkForm < Hyrax::Forms::WorkForm
    self.model_class = ::GenericWork
    self.terms += [:resource_type, :contact_email]
    self.required_fields += [:contact_email]
    self.required_fields -= [:keyword, :rights]
  end
end
```

### Default form field behavior

Default behavior:
- By adding the property to self.terms, it will be added to the new/edit form.
- Without additional customization, the field will be a text input field.  (See [Hyrax's app/views/records/edit_fields/_default.html.erb](https://github.com/projecthydra/hyrax/blob/master/app/views/records/edit_fields/_default.html.erb)
- Because we did not set multiple: true, there will be only a single value set for this property.
- Because we added it to the required_fields set, it will be displayed as required on the initial display of metadata fields on the form. If you did not add it to the set of required_fields, it will display in the form when you click Additional Fields button.

### Customizing the form field

Optionally, you can customize the creation of the form field.  To customize a form field, you create a partial with the property name under app/views/records/edit_fields.  Add form code to display the form as desired.  If this is the first form field customization you have made, you will need to create the `app/views/records/edit_fields` directories under `app/views`.

For a single-value field, you can use something similar to...
```erb
<% # app/views/records/edit_fields/_contact_email.html.erb %>
<%= f.input :contact_email, as: :email, required: f.object.required?(key),
  input_html: { class: 'form-control', multiple: false }
%>
```

You can see [more examples](https://github.com/projecthydra/hyrax/tree/master/app/views/records/edit_fields) by exploring those created for the default fields in Hyrax.

## Add the new single-value property to the show page

By default, the new property will **NOT** be displayed on the show page for works of this type.

- See [Curation Concern's work_show_presenter.rb](https://github.com/projecthydra/curation_concerns/blob/master/app/presenters/curation_concerns/work_show_presenter.rb) around line 39 with comment # Metadata Methods to see the primary list of properties that will be displayed on the show page.
- See [Hyrax's work_show_presenter.rb](https://github.com/projecthydra/hyrax/blob/master/app/presenters/hyrax/work_show_presenter.rb) for a few additional properties that are displayed on the work show page.  Look for the property names delegated to the solr_document near the top of the file.

### Define the class to use to control the show page

Display of the show page for a work is controlled by a presenter class specified in the controller.  This is not automatically generated and set by the generator process.  By default, [Hyrax's work_show_presenter.rb](https://github.com/projecthydra/hyrax/blob/master/app/presenters/hyrax/work_show_presenter.rb) class is used for show pages.

Create the following as a starting point for the custom presenter class.

```ruby
# app/presenters/generic_work_presenter.rb
class GenericWorkPresenter < Hyrax::WorkShowPresenter
end
```

Assign the presenter class in the generated controller.  Edit `app/controllers/curation_concerns/generic_work_controller.rb` and add the following line under the curation_concern_type assignment.

```ruby
  self.show_presenter = GenericWorkPresenter
```

The controller class now looks like...

```ruby
module CurationConcerns
  class GenericWorksController < ApplicationController
    include CurationConcerns::CurationConcernController
    # Adds Hyrax behaviors to the controller.
    include Hyrax::WorksControllerBehavior

    self.curation_concern_type = GenericWork
    self.show_presenter = GenericWorkPresenter
  end
end
```

### Add the property to be displayed

Edit the custom presenter class (e.g. app/presenters/generic_work_presenter.rb) and add a delegate to solr_document for the property to be displayed.

```ruby
  delegate :contact_email, to: :solr_document
```

The full custom presenter class now looks like...
```ruby
# app/presenters/generic_work_presenter.rb
class GenericWorkPresenter < Hyrax::WorkShowPresenter
  delegate :contact_email, to: :solr_document
end
```

Edit app/models/solr_document.rb and add a method to retrieve the property's value from the solr doc.
```ruby
def contact_email
  self[Solrizer.solr_name('contact_email')]
end
```

If this is the first custom property added to the show page, you will need to copy [Hyrax's app/views/curation_concerns/base/_attribute_rows.html.erb](https://github.com/projecthydra/hyrax/blob/master/app/views/curation_concerns/base/_attribute_rows.html.erb) to the same directory structure in your app.  NOTE: The link goes to master.  Make sure you copy from the release/branch of Hyrax that your app has installed.

Add the field to the local copy of app/views/curation_concerns/base/_attribute_rows.html.erb
```erb
<%= presenter.attribute_to_html(:contact_email) %>
```

### Configure Blacklight to show the property on the show page

Edit app/controllers/catalog_controller.rb and look for the section including `add_show_field` statements.  Add the following:
```ruby
config.add_show_field solr_name("contact_email", :stored_searchable), label: "Contact Email"
```

### Configure Blacklight to show the property in search results

Optionally, you can also have the property shown in the search results for a work.

Edit app/controllers/catalog_controller.rb and look for the section including `add_index_field` statements.  Add the following:
```ruby
config.add_index_field solr_name("contact_email", :stored_searchable), label: "Contact Email"
```

### Default property display behavior

Each value for the property, in this case the single value, will be displayed using a `<span>` tag.  See [Curation Concerns's default AttributeRenderer](https://github.com/projecthydra/curation_concerns/blob/master/app/renderers/curation_concerns/renderers/attribute_renderer.rb).

### Customizing the property display

Optionally, you can customize the display of the property value on the show page.

Either use an existing renderer or define a new renderer.  The renderer can be for a type of attribute, like an email, that can be used with multiple properties or a one-off renderer for a specific property.  The process is the same for both.

To define a general renderer for all email properties...
```erb
# app/renderers/email_attribute_renderer.rb
class EmailAttributeRenderer < CurationConcerns::Renderers::AttributeRenderer
  def attribute_value_to_html(value)
    %(<span itemprop="email"><a href="mailto:#{value}">#{value}</a></span>)
  end
end
```

Identify the renderer to use for the property.  Edit the definition in the local copy of app/views/curation_concerns/base/_attribute_rows.html.erb and add the render_as parameter for the property.
```erb
<%= presenter.attribute_to_html(:contact_email, render_as: :email) %>
```

NOTES:
* The class name must begin with the renderer name and end with AttributeRenderer.
* To identify a renderer, use the renderer name (everything upto, but not including AttributeRenderer)
* You can use one of the renderers defined in Curation Concerns.
* You can make more complex renderers.  See Curation Concerns defined renderers for examples.
* See [Curation Concerns defined renderers](https://github.com/projecthydra/curation_concerns/tree/master/app/renderers/curation_concerns/renderers).

---
# Customizing the work-type to include a multi-value text property

Reference [Customizing the work-type to include a single-value text property](#customizing-the-work-type-to-include-a-single-value-text-property) section for the overall process.  This section includes only new changes to support multi-value properties.

## Add the new multi-value property to the model

The GenericWork class after the customization for the single-value property looked like...

```ruby
class GenericWork < ActiveFedora::Base
  include ::CurationConcerns::WorkBehavior
  include ::CurationConcerns::BasicMetadata
  include Hyrax::WorkBehavior
  self.human_readable_type = 'Generic Work'
  # Change this to restrict which works can be added as a child.
  # self.valid_child_concerns = []
  validates :title, presence: { message: 'Your work must have a title.' }

  property :contact_email, predicate: ::RDF::Vocab::VCARD.hasEmail, multiple: false do |index|
    index.as :stored_searchable
  end
end
```

### Extend the model to add a new multi-value property

To define a property that has multiple text values, add the following to the GenericWork model.

```ruby
  property :contact_phone, predicate: ::RDF::Vocab::VCARD.hasTelephone do |index|
    index.as :stored_searchable
  end
```

Expected behaviors for this property:
- Can have one or more values assigned.  NOTE: By default properties are multi-value.  You can also explicitly state this by adding `, multiple: true` before `do |index|`
- The remaining basic behaviors are the same as for single-value properties.  See more information under [Add the new single-value property to the model](#add-the-new-single-value-property-to-the-model) Expected behaviors.

### The modified model

The GenericWork at this point with the single and multi-valued properties looks like...

```ruby
class GenericWork < ActiveFedora::Base
  include ::CurationConcerns::WorkBehavior
  include ::CurationConcerns::BasicMetadata
  include Hyrax::WorkBehavior
  self.human_readable_type = 'Generic Work'
  # Change this to restrict which works can be added as a child.
  # self.valid_child_concerns = []
  validates :title, presence: { message: 'Your work must have a title.' }

  property :contact_email, predicate: ::RDF::Vocab::VCARD.hasEmail, multiple: false do |index|
    index.as :stored_searchable
  end

  property :contact_phone, predicate: ::RDF::Vocab::VCARD.hasTelephone do |index|
    index.as :stored_searchable
  end
end
```

## Add the new multi-value property to the new/edit form

The GenericWorkForm class after the customization for the single-value property looked like...

```ruby
# app/forms/generic_work_form.rb
# Generated via
#  `rails generate curation_concerns:work GenericWork`
module CurationConcerns
  class GenericWorkForm < Hyrax::Forms::WorkForm
    self.model_class = ::GenericWork
    self.terms += [:resource_type, :contact_email]
    self.required_fields += [:contact_email]
    self.required_fields -= [:keyword, :rights]
  end
end
```

See [Add the new single-value property to the new/edit form](#add-the-new-single-value-property-to-the-newedit-form) section for more information on the GenericWorkForm class.

### Adding the property to the work-type's new/edit form

Now we want to update GenericWorkForm to include the new multi-value property.

Modify `self.terms` to include the new property.

```ruby
    self.terms += [:resource_type, :contact_email, :contact_phone]
```

The full class after the changes looks like...
```ruby
# app/forms/generic_work_form.rb
# Generated via
#  `rails generate curation_concerns:work GenericWork`
module CurationConcerns
  class GenericWorkForm < Hyrax::Forms::WorkForm
    self.model_class = ::GenericWork
    self.terms += [:resource_type, :contact_email, :contact_phone]
    self.required_fields += [:contact_email]
    self.required_fields -= [:keyword, :rights]
  end
end
```

See [Add the new single-value property to the new/edit form](#add-the-new-single-value-property-to-the-newedit-form) section for more information on modifying required_fields.

### Default form field behavior

Default behavior:
- By adding the property to self.terms, it will be added to the new/edit form.
- Without additional customization, the field will be a text input field.  (See [Hyrax's app/views/records/edit_fields/_default.html.erb](https://github.com/projecthydra/hyrax/blob/master/app/views/records/edit_fields/_default.html.erb)
- Because we did not set a value for multiple:, there can be one or more values set for this property.
- Because we did not add it to the required_fields set, it will be displayed in the form when you click Additional Fields button.

### Customizing the form field

Optionally, you can customize the creation of the form field.  To customize a form field, you create a partial with the property name under app/views/records/edit_fields.  Add form code to display the form as desired.

For a multi-value field, you can use something similar to...
```erb
<% # app/views/records/edit_fields/_contact_phone.html.erb %>
<%= f.input :contact_phone, as: :tel, required: f.object.required?(key),
    input_html: { class: 'form-control', multiple: true }
%>
```

You can see [more examples](https://github.com/projecthydra/hyrax/tree/master/app/views/records/edit_fields) by exploring those created for the default fields in Hyrax.

## Add the new multi-value property to the show page

By default, the new property will **NOT** be displayed on the show page for works of this type.

### Define the class to use to control the show page

A presenter class is **NOT** generated as part of the generation process, but we created one for the single-value property.

The GenericWorkPresenter class after the customization for the single-value property looked like...

```ruby
# app/presenters/generic_work_presenter.rb
class GenericWorkPresenter < Hyrax::WorkShowPresenter
  delegate :contact_email, to: :solr_document
end
```

See [Add the new single-value property to the show page](#add-the-new-single-value-property-to-the-show-page) section for more information.

### Add the property to be displayed

Edit the custom presenter class (e.g. app/presenters/generic_work_presenter.rb) and add a delegate to solr_document for the property to be displayed.

```ruby
  delegate :contact_email, :contact_phone, to: :solr_document
```

The full custom presenter class now looks like...
```ruby
# app/presenters/generic_work_presenter.rb
class GenericWorkPresenter < Hyrax::WorkShowPresenter
  delegate :contact_email, :contact_phone, to: :solr_document
end
```

Edit app/models/solr_document.rb and add a method to retrieve the property's value from the solr doc.
```ruby
def contact_email
  self[Solrizer.solr_name('contact_email')]
end

def contact_phone
  self[Solrizer.solr_name('contact_phone')]
end
```

Add the field to the local copy of app/views/curation_concerns/base/_attribute_rows.html.erb  If this file doesn't exist in your local app, see [Add the new single-value property to the show page](#add-the-new-single-value-property-to-the-show-page) for where to get a copy.
```erb
<%= presenter.attribute_to_html(:contact_email) %>
<%= presenter.attribute_to_html(:contact_phone) %>
```

### Configure Blacklight to show the property on the show page

Edit app/controllers/catalog_controller.rb and look for the section including add_show_field statements.  Add the following:
```ruby
config.add_show_field solr_name("contact_email", :stored_searchable), label: "Contact Email"
config.add_show_field solr_name("contact_phone", :stored_searchable), label: "Contact Phone"
```

### Configure Blacklight to show the property in search results

Optionally, you can also have the property shown in the search results for a work.

Edit app/controllers/catalog_controller.rb and look for the section including `add_index_field` statements.  Add the following:
```ruby
config.add_index_field solr_name("contact_email", :stored_searchable), label: "Contact Email"
config.add_index_field solr_name("contact_phone", :stored_searchable), label: "Contact Phone"
```

### Default property display behavior

Each value for the property will be displayed using a `<span>` tag.  See [Curation Concerns's default AttributeRenderer](https://github.com/projecthydra/curation_concerns/blob/master/app/renderers/curation_concerns/renderers/attribute_renderer.rb).

### Customizing the property display

Optionally, you can customize the display of the property value on the show page.

Either use an existing renderer or define a new renderer.  The renderer can be for a type of attribute, like a phone, that can be used with multiple properties or a one-off renderer for a specific property.  The process is the same for both.

To define a general renderer for all phone properties...
```erb
# app/renderers/phone_attribute_renderer.rb
class PhoneAttributeRenderer < CurationConcerns::Renderers::AttributeRenderer
  def attribute_value_to_html(value)
    %(<span itemprop="telephone"><a href="tel:#{value}">#{value}</a></span>)
  end
end
```

Identify the renderer to use for the property.  Edit the definition in the local copy of app/views/curation_concerns/base/_attribute_rows.html.erb and add the render_as parameter for the property.
```erb
<%= presenter.attribute_to_html(:contact_phone, render_as: :phone) %>
```

See [Add the new single-value property to the show page](#add-the-new-single-value-property-to-the-show-page) Customizing the property display section for more information.

---
# Customizing the work-type to include a controlled vocabulary property

Reference [Customizing the work-type to include a single-value text property](#customizing-the-work-type-to-include-a-single-value-text-property) section for the overall process.  This section includes only new changes to support controlled-value properties.

## Add the new controlled-value property to the model

The GenericWork class after the customization for the multi-value property looked like...

```ruby
class GenericWork < ActiveFedora::Base
  include ::CurationConcerns::WorkBehavior
  include ::CurationConcerns::BasicMetadata
  include Hyrax::WorkBehavior
  self.human_readable_type = 'Generic Work'
  # Change this to restrict which works can be added as a child.
  # self.valid_child_concerns = []
  validates :title, presence: { message: 'Your work must have a title.' }

  property :contact_email, predicate: ::RDF::Vocab::VCARD.hasEmail, multiple: false do |index|
    index.as :stored_searchable
  end

  property :contact_phone, predicate: ::RDF::Vocab::VCARD.hasTelephone do |index|
    index.as :stored_searchable
  end
end
```

### Extend the model to add a new controlled-value property

The controlled-value property based on values in a controlled vocabulary is defined in the same was as the single and multi-valued properties.

To define a property with values driven from a controlled vocabulary is initially the same.  Add the following to the GenericWork model.
```ruby
  property :department, predicate: ::RDF::URI.new("http://lib.my.edu/departments"), multiple: false do |index|
    index.as :stored_searchable, :facetable
  end
```

### The modified model

With all three properties added, the GenericWork now looks like:

```ruby
class GenericWork < ActiveFedora::Base
  include ::CurationConcerns::WorkBehavior
  include ::CurationConcerns::BasicMetadata
  include Hyrax::WorkBehavior
  self.human_readable_type = 'Generic Work'
  # Change this to restrict which works can be added as a child.
  # self.valid_child_concerns = []
  validates :title, presence: { message: 'Your work must have a title.' }

  property :contact_email, predicate: ::RDF::Vocab::VCARD.hasEmail, multiple: false do |index|
    index.as :stored_searchable
  end

  property :contact_phone, predicate: ::RDF::Vocab::VCARD.hasTelephone do |index|
    index.as :stored_searchable
  end

  property :department, predicate: ::RDF::URI.new("http://lib.my.edu/departments"), multiple: false do |index|
    index.as :stored_searchable, :facetable
  end
end
```

## Add the new controlled-value property to the new/edit form

The GenericWorkForm class after the customization for the multi-value property looked like...

```ruby
# Generated via
#  `rails generate curation_concerns:work GenericWork`
module CurationConcerns
  class GenericWorkForm < Hyrax::Forms::WorkForm
    self.model_class = ::GenericWork
    self.terms += [:resource_type, :contact_email, :contact_phone]
    self.required_fields += [:contact_email]
    self.required_fields -= [:keyword, :rights]
  end
end
```

See [Add the new single-value property to the new/edit form](#add-the-new-single-value-property-to-the-newedit-form) section for more information on the GenericWorkForm class.

### Adding the property to the work-type's new/edit form

Now we want to update GenericWorkForm to include the new controlled-value property.

Modify `self.terms` to include the new property.

```ruby
    self.terms += [:resource_type, :contact_email, :contact_phone, :department]
```

The full class after the changes looks like...
```ruby
# app/forms/generic_work_form.rb
# Generated via
#  `rails generate curation_concerns:work GenericWork`
module CurationConcerns
  class GenericWorkForm < Hyrax::Forms::WorkForm
    self.model_class = ::GenericWork
    self.terms += [:resource_type, :contact_email, :contact_phone, :department]
    self.required_fields += [:contact_email]
    self.required_fields -= [:keyword, :rights]
  end
end
```

See [Add the new single-value property to the new/edit form](#add-the-new-single-value-property-to-the-newedit-form) section for more information on modifying required_fields.

### Default form field behavior

Default behavior will be the same for the single-value property at this point which is the creation of a text input field for the user to type in the value.  But that's not what we want in this case.

### Create a vocabulary

Before we try to use the controlled vocabulary, we need to create it first.
```ruby
## config/authorities/departments.yml
terms:
  - id: eng
    term: English
  - id: hst
    term: History
  - id: ltn
    term: Latin
  - id: zoo
    term: Zoology
```

NOTE: Support for controlled vocabularies is provided by the [Questioning Authority](https://github.com/projecthydra/questioning_authority) (qa) gem.  See that gem for more details on controlled vocabularies.

### Create a service to load the vocabulary

A service is required to set up Questioning Authority to return all the values, which will be used to populate the selection list, and a single value given an id, which will be used to show the value instead of the id on the show page.

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

### Customizing the form field

For a controlled vocabulary, you must customize the creation of the form field.  To customize a form field, you create a partial with the property name under app/views/records/edit_fields.  Add form code to display the form as desired.

For a controlled-value field, you can use something similar to...

```erb
# app/views/records/edit_fields/_department.html.erb
<%= f.input :department, as: :select,
    collection: DepartmentsService.select_all_options,
    include_blank: true,
    item_helper: method(:include_current_value),
    input_html: { class: 'form-control' }
%>
```

You can see [more examples](https://github.com/projecthydra/hyrax/tree/master/app/views/records/edit_fields) by exploring those created for the default fields in Hyrax.

## Add the new controlled-value property to the show page

By default, the new property will **NOT** be displayed on the show page for works of this type.

### Define the class to use to control the show page

A presenter class is **NOT** generated as part of the generation process, but we created one for the single-value property.

The GenericWorkPresenter class after the customization for the multi-value property looked like...

```ruby
# app/presenters/generic_work_presenter.rb
class GenericWorkPresenter < Hyrax::WorkShowPresenter
  delegate :contact_email, :contact_phone, to: :solr_document
end
```

See [Add the new single-value property to the show page](#add-the-new-single-value-property-to-the-show-page) section for more information.

### Add the property to be displayed

Edit the custom presenter class (e.g. app/presenters/generic_work_presenter.rb) and add a delegate to solr_document for the property to be displayed.

```ruby
  delegate :contact_email, :contact_phone, :department, to: :solr_document
```

The full custom presenter class now looks like...
```ruby
# app/presenters/generic_work_presenter.rb
class GenericWorkPresenter < Hyrax::WorkShowPresenter
  delegate :contact_email, :contact_phone, :department, to: :solr_document
end
```

Edit app/models/solr_document.rb and add a method to retrieve the property's value from the solr doc.
```ruby
def contact_email
  self[Solrizer.solr_name('contact_email')]
end

def contact_phone
  self[Solrizer.solr_name('contact_phone')]
end

def department
  self[Solrizer.solr_name('department')]
end
```

Add the field to the local copy of app/views/curation_concerns/base/_attribute_rows.html.erb  If this file doesn't exist in your local app, see [Add the new single-value property to the show page](#add-the-new-single-value-property-to-the-show-page) for where to get a copy.
```erb
<%= presenter.attribute_to_html(:contact_email) %>
<%= presenter.attribute_to_html(:contact_phone) %>
<%= presenter.attribute_to_html(:department) %>
```

### Configure Blacklight to show the property on the show page

Edit app/controllers/catalog_controller.rb and look for the section including add_show_field statements.  Add the following:
```ruby
config.add_show_field solr_name("contact_email", :stored_searchable), label: "Contact Email"
config.add_show_field solr_name("contact_phone", :stored_searchable), label: "Contact Phone"
config.add_show_field solr_name("department", :stored_searchable), label: "Department"
```

### Configure Blacklight to show the property in search results

Optionally, you can also have the property shown in the search results for a work.

Edit app/controllers/catalog_controller.rb and look for the section including `add_index_field` statements.  Add the following:
```ruby
config.add_index_field solr_name("contact_email", :stored_searchable), label: "Contact Email"
config.add_index_field solr_name("contact_phone", :stored_searchable), label: "Contact Phone"
config.add_index_field solr_name("department", :stored_searchable), label: "Department"
```

### Default property display behavior

Each value for the property, in this case the selected controlled value's ID, will be displayed using a `<span>` tag.  See [Curation Concerns's default AttributeRenderer](https://github.com/projecthydra/curation_concerns/blob/master/app/renderers/curation_concerns/renderers/attribute_renderer.rb).

NOTE: If the ID and TERM for your controlled vocabulary are the same, then you can use the default display behavior.

### Customizing the property display

For a controlled vocabulary, you must customize the display of the property on the show page if the ID is not the same as the TERM; otherwise, the ID will be displayed instead of the value.

Define a new renderer to convert the value from the controlled value's ID to its TERM.  The renderer in this case will be specific to the controlled value property.

To define a property specific renderer for the department property...
```erb
# app/renderers/department_attribute_renderer.rb
class DepartmentAttributeRenderer < CurationConcerns::Renderers::AttributeRenderer
  def attribute_value_to_html(value)
    %(<span itemprop="department">#{::DepartmentsService.label(value)}</span>)
  end
end

```

Identify the renderer to use for the property.  Edit the definition in the local copy of app/views/curation_concerns/base/_attribute_rows.html.erb and add the render_as parameter for the property.
```erb
<%= presenter.attribute_to_html(:department, render_as: :department) %>
```

See [Add the new single-value property to the show page](#add-the-new-single-value-property-to-the-show-page) Customizing the property display section for more information.

---
# Modifying default Hyrax fields

## Remove a default property from the set of required fields

Edit app/forms/generic_work_form.rb  (substitute your work-type name for generic_work) and make add the following to make keyword and rights optional fields.  NOTE: This also moves these fields below all required fields and they only display on the form when the Additional Fields button is clicked.

```ruby
    self.required_fields -= [:keyword, :rights]
```

## Making a default property non-repeatable

By default all fields in Hyrax are repeatable. If you'd like to change this behavior for a field that Hyrax provides out of the box, you can do the following.  This example makes title, description, and publisher fields single-value.

Edit app/forms/generic_work_form.rb  (substitute your work-type name for generic_work) and make the following changes

* Override `self.multiple?(field)` and return false for any default fields you want to be single value.
* Override `self.model_attributes(_)` to cast back to multi-value when saving
* Add methods to return the field value as single-value for populating the form fields during editing

The form class after making these changes looks like...

```ruby
# app/forms/generic_work_form.rb
# Generated via
#  `rails generate curation_concerns:work GenericWork`
module CurationConcerns
  class GenericWorkForm < Hyrax::Forms::WorkForm
    self.model_class = ::GenericWork

    def self.multiple?(field)
      if [:title, :description, :publisher].include? field.to_sym
        false
      else
        super
      end
    end

    def self.model_attributes(_)
      attrs = super
      attrs[:title] = Array(attrs[:title]) if attrs[:title]
      attrs[:description] = Array(attrs[:description]) if attrs[:description]
      attrs[:publisher] = Array(attrs[:publisher]) if attrs[:publisher]
      attrs
    end

    def title
      super.first || ""
    end

    def description
      super.first || ""
    end

    def publisher
      super.first || ""
    end
  end
end
```

# Creating a Default Deposit Agreement

By default, Hyrax will ask you to accept a deposit agreement each time you upload a file. You can make this implicit by having a passive agreement instead. To do this, change the `app/config/initializers/hyrax.rb` to:

``` ruby
  config.active_deposit_agreement_acceptance = false
```

Create custom translations in your `hyrax.en.yml` locales file:

``` yml
en:
  hyrax:
    passive_consent_to_agreement: "By clicking the Save button, I am agreeing to etc..."
    deposit_agreement: "Institutional Agreement"
```

Lastly, create your own `app/views/static/agreement.html.erb` page with the content of your deposit agreement.

# Customizing display of Collection properties

The collection properties do not use the renderer process to control the display of properties.

To modify the display of a colleciton property...
* add a partial with the property's name to app/views/records/show_fields  (e.g. _department.html.erb)
* in that file, include markup to control the display of the field

NOTE: See [Hyrax's show_fields](https://github.com/projecthydra/hyrax/tree/master/app/views/records/show_fields) for examples.
