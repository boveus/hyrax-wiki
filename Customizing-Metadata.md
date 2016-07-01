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

# Override the GenericWork model

The GenericWork class is provided by Sufia, but we want to update the model with our own metadata so we define it in our app to override what Sufia provides. Since Rails finds the class in our local application, it won't load it from Sufia.

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
We'll add a select box of university departments, and make it a non-repeatable field.
Add the property declaration into the `GenericWork` class (and pass a block to make sure the property is indexed in Solr):

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
# services/departments_services.rb
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
# records/edit_fields/_.department.html.erb
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
    self.required_fields +=[:department] # Add this line if you'd like to make your field required

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

# Note

Please note that the documentation below applies to Sufia <7.

Table of Contents
=================

  * [Override the GenericFile model](#override-the-genericfile-model)
    * [Add the property](#add-the-property)
    * [Field repeatability](#field-repeatability)
    * [Try the property at the Rails console](#try-the-property-at-the-rails-console)
  * [Extend the presenter](#extend-the-presenter)
    * [Required fields](#required-fields)
    * [Set the controller to use our presenter](#set-the-controller-to-use-our-presenter)
  * [Create edit forms](#create-edit-forms)
    * [Set the controllers to use our forms](#set-the-controllers-to-use-our-forms)
  * [Labels and help text](#labels-and-help-text)

# Override the GenericFile model

The GenericFile class is provided by Sufia, but we want to update the model with our own metadata so we define it in our app to override what Sufia provides. Since Rails finds the class in our local application, it won't load it from Sufia.

```ruby
# app/models/generic_file.rb
class GenericFile < ActiveFedora::Base
  include Sufia::GenericFile
end
```

## Add the property

Add the property declaration into the `GenericFile` class (and pass a block to make sure the property is indexed in Solr):

```ruby
  property :alternative, predicate: ::RDF::DC.alternative do |index|
    index.as :stored_searchable, :facetable
  end
```

When you're done the file should look like this:
```ruby
# app/models/generic_file.rb
class GenericFile < ActiveFedora::Base
  include Sufia::GenericFile

  property :alternative, predicate: ::RDF::DC.alternative do |index|
    index.as :stored_searchable, :facetable
  end
end
```

## Field repeatability

By default, properties are repeatable.  If you want the property to be single-valued rather than repeatable, then you would add `multiple: false` to its property definition, as follows:

```ruby
  property :alternative, predicate: ::RDF::DC.alternative, multiple: false do |index|
    index.as :stored_searchable, :facetable
  end
```


## Try the property at the Rails console

You can try using the new property at the rails console like this. I've shown the expected output after each command.
```
$ rails c
Loading development environment (Rails 4.1.8)
irb(main):001:0> f = GenericFile.new(id: 'test-1')
  LocalAuthority Load (1.2ms)  SELECT  "local_authorities".* FROM "local_authorities"  WHERE "local_authorities"."name" = 'lc_subjects' LIMIT 1
  LocalAuthority Load (1.4ms)  SELECT  "local_authorities".* FROM "local_authorities"  WHERE "local_authorities"."name" = 'lexvo_languages' LIMIT 1
  LocalAuthority Load (1.3ms)  SELECT  "local_authorities".* FROM "local_authorities"  WHERE "local_authorities"."name" = 'lc_genres' LIMIT 1
ActiveFedora: loading fedora config from /opt/sufia/config/fedora.yml
ActiveFedora: loading solr config from /opt/sufia/config/solr.yml
Attempted to init base path `dev`, but it already exists
=> #<GenericFile id: "test-1", mime_type: "", format_label: [""], file_size: [""], last_modified: [""], filename: [""], original_checksum: [""], rights_basis: [], copyright_basis: [], copyright_note: [], well_formed: [], valid: [""], status_message: [], file_title: [""], file_author: [""], page_count: [""], file_language: [], word_count: [], character_count: [], paragraph_count: [], line_count: [], table_count: [], graphics_count: [], byte_order: [], compression: [], color_space: [], profile_name: [], profile_version: [], orientation: [], color_map: [], image_producer: [], capture_device: [], scanning_software: [], exif_version: [], gps_timestamp: [], latitude: [], longitude: [], character_set: [], markup_basis: [], markup_language: [], bit_depth: [], channels: [], data_format: [], offset: [], frame_rate: [], label: nil, depositor: nil, relative_path: nil, import_url: nil, part_of: [], resource_type: [], title: [], creator: [], contributor: [], description: [], tag: [], rights: [], publisher: [], date_created: [], date_uploaded: nil, date_modified: nil, subject: [], language: [], identifier: [], based_near: [], related_url: [], bibliographic_citation: [], source: [], proxy_depositor: nil, on_behalf_of: nil, alternative: [], batch_id: nil>
irb(main):002:0> f.alternative = ["An alternative title"]
=> ["An alternative title"]
irb(main):003:0> puts f.resource.dump(:ttl)

<http://127.0.0.1:8983/fedora/rest/dev/te/st/-1/test-1> <http://purl.org/dc/terms/alternative> "An alternative title";
   <info:fedora/fedora-system:def/model#hasModel> "GenericFile" .
=> nil
irb(main):004:0>
```

# Extend the presenter

Sufia provides default presenter classes to control what properties of a model are display in your app, and in what order they appear. We create a subclass of Sufia's presenter adding `alternative` to the list of terms.

```ruby
# app/presenters/my_generic_file_presenter.rb
class MyGenericFilePresenter < Sufia::GenericFilePresenter
  self.terms = [:resource_type, :title, :creator, :contributor, :description,
                :tag, :rights, :publisher, :date_created, :subject, :language,
                :identifier, :based_near, :related_url, :alternative]
end
```

## Set the controller to use our presenter

Now we override Sufia's GenericFilesController so that it uses the MyGenericFilePresenter class that we defined rather than the default presenter in Sufia.

```ruby
# app/controllers/generic_files_controller.rb
class GenericFilesController < ApplicationController
  include Sufia::Controller
  include Sufia::FilesControllerBehavior

  self.presenter_class = MyGenericFilePresenter
end
```

The fields show up in the show view of our app, but we also want them on our edit form too. Let's create an edit form that has our new field.

# Create edit forms

Sufia also provides default form classes to control what properties of a model are editable in your app, and in what order they appear. Form classes extend presenter classes to include permissions and required fields, so we can base this work on what we already did with `MyGenericFilePresenter`. We subclass our own custom presenter to pick up the changes we already made there.

```ruby
# app/forms/my_file_edit_form.rb
class MyFileEditForm < MyGenericFilePresenter
  include HydraEditor::Form
  include HydraEditor::Form::Permissions
end
```

On initial upload, Sufia allows users to edit metadata in bulk using a batch edit form, so we need to create a second form to be used by the `BatchController`.

```ruby
# app/forms/my_batch_edit_form.rb
class MyBatchEditForm < MyFileEditForm
end
```

## Required fields

We can also give `MyFileEditForm` a different set of required and non-required fields to override what it inherits from Sufia's GenericFilePresenter class:

```ruby
# app/forms/my_file_edit_form.rb
class MyFileEditForm < MyGenericFilePresenter
  include HydraEditor::Form
  include HydraEditor::Form::Permissions

  self.required_fields = [:title, :alternative, :creator, :tag, :rights]
end
```

In theory, we can give `MyBatchEditForm` its own `required_fields` value to override the value it inherits from `MyFileEditForm`, but this might be something to think twice before doing.

## Set the controllers to use our forms

Now we override Sufia's GenericFilesController and BatchController so that they use our custom edit form classes rather than the defaults in Sufia.

Add the following line to `app/controllers/generic_files_controller.rb`

```ruby
  self.edit_form_class = MyFileEditForm
```

The whole file should look like this:

```ruby
# app/controllers/generic_files_controller.rb
class GenericFilesController < ApplicationController
  include Sufia::Controller
  include Sufia::FilesControllerBehavior

  self.presenter_class = MyGenericFilePresenter
  self.edit_form_class = MyFileEditForm
end
```

And add the following line to `app/controllers/batch_controller.rb`

```ruby
  self.edit_form_class = MyBatchEditForm
```

The whole file should look like this:

```ruby
# app/controllers/batch_controller.rb
class BatchController < ApplicationController
  include Sufia::BatchControllerBehavior

  self.edit_form_class = MyBatchEditForm
end
```

The batch edits controller was not modified to allow you to set a form the way the batch controller was. (Batch edits are for editing from the dashboard; batch is about uploading groups of files together). So you have to override #terms directly. Add `app/controllers/batch_edits_controller.rb`:

```ruby
class BatchEditsController < ApplicationController
  include Hydra::BatchEditBehavior
  include GenericFileHelper
  include Sufia::BatchEditsControllerBehavior

  def terms
    MyBatchEditForm.terms
  end
end
```

It would be nice to make things more consistent here; note this may all be overhauled soon due to pcdm.

# Labels and help text

By default the label for the field in the form is taken from the property label, with the initial letter capitalised. Here is an example of customising it, and the associated help text:

```ruby
# config/locales/sufia.en.yml
en:
  simple_form:
    labels:
      generic_file:
        alternative: "Alternative Title"
    metadata_help:
      generic_file:
        alternative: "An alternative name for a file."
```