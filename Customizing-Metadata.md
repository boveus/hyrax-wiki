### Create the GenericFile model
the GenericFile class is typically provided by Sufia, but we want to update the model, so we define it in our app instead. Since Rails finds the class in our local application, it won't load it from Sufia.
```ruby
# app/models/generic_file.rb
class GenericFile < ActiveFedora::Base
  include Sufia::GenericFile
end
```

### Add the property
Add the property declaration into the `GenericFile` class:
```ruby
  property :alternative, predicate: ::RDF::DC.alternative
```

When you're done the file should look like this:
```ruby
# app/models/generic_file.rb
class GenericFile < ActiveFedora::Base
  include Sufia::GenericFile
  property :alternative, predicate: ::RDF::DC.alternative
end
```


### Extend the presenter
We create a subclass of the presenter adding `alternative` to the list of terms
```ruby
# app/presenters/my_generic_file_presenter.rb
class MyGenericFilePresenter < Sufia::GenericFilePresenter
  self.terms = [:resource_type, :title, :creator, :contributor, :description,
                       :tag, :rights, :publisher, :date_created, :subject, :language,
                       :identifier, :based_near, :related_url, :alternative]
end
```

### Set the controller to use our presenter
```ruby
# app/controllers/generic_files_controller.rb
class GenericFilesController < ApplicationController
  include Sufia::Controller
  include Sufia::FilesControllerBehavior
  self.presenter_class = MyGenericFilePresenter
end
```

Now the fields show up in the show view, but we also want them on our edit form too. Let's create an edit form that has our field

### Create forms
This form will extend the presenter we already created.

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