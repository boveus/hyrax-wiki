### Override the GenericFile model

The GenericFile class is provided by Sufia, but we want to update the model with our own metadata so we define it in our app to override what Sufia provides. Since Rails finds the class in our local application, it won't load it from Sufia.

```ruby
# app/models/generic_file.rb
class GenericFile < ActiveFedora::Base
  include Sufia::GenericFile
end
```

### Add the property

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


### Extend the presenter

Sufia provides default presenter classes to control what properties of a model are display in your app, and in what order they appear. We create a subclass of Sufia's presenter adding `alternative` to the list of terms.

```ruby
# app/presenters/my_generic_file_presenter.rb
class MyGenericFilePresenter < Sufia::GenericFilePresenter
  self.terms = [:resource_type, :title, :creator, :contributor, :description,
                       :tag, :rights, :publisher, :date_created, :subject, :language,
                       :identifier, :based_near, :related_url, :alternative]
end
```

### Set the controller to use our presenter

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

### Create forms

Sufia also provides default form classes to control what properties of a model are editable in your app, and in what order they appear. Form classes extend presenter classes to include permissions and required fields, so we can base this work on what we already did with `MyGenericFilePresenter`. We subclass our own custom presenter to pick up the changes we already made there.

```ruby
# app/forms/my_file_edit_form.rb
class MyFileEditForm < MyGenericFilePresenter
  include HydraEditor::Form
  include HydraEditor::Form::Permissions

  self.required_fields = [:title, :creator, :tag, :rights]
end
```

Create a batch form
```ruby
# app/forms/my_batch_edit_form.rb
class MyBatchEditForm < MyFileEditForm
end
```

### Set the controller to use our form
Add the following line to `app/controllers/generic_files_controller.rb`
```ruby
  self.edit_form_class = MyFileEditForm
```

The whole file should look like this:
```ruby
class GenericFilesController < ApplicationController
  include Sufia::Controller
  include Sufia::FilesControllerBehavior
  self.presenter_class = MyGenericFilePresenter
  self.edit_form_class = MyFileEditForm
end
```

And add the following line to app/controllers/batch_controller.rb
```ruby
  self. edit_form_class = MyFileEditForm
```

The whole file should look like this:
```ruby
# app/controllers/batch_controller.rb
class BatchController < ApplicationController
  include Sufia::BatchControllerBehavior
  self.edit_form_class = MyBatchEditForm
end
```

# Required/non-required fields

`TODO`

# Field repeatability

`TODO`

# Labels and help text

By default the label for the field in the form is taken from the property label, with the initial letter capitalised.  Here is an example of customising it, and the associated help text:

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

`TODO` (I assume there is a different way of handling the label in the batch form?)