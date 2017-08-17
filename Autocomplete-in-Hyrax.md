# Hyrax Autocomplete

## Activating autocomplete on a field

The way a field is associated with a particular autocomplete data source is with a data attribute on the element. The `data-autocomplete-url` attribute stores a path to the data source. 
To add one to an existing field create a partial for the field in `app/views/records/edit_fields/`. 

In the partial, specify the data source: 

```erb
<%=
  f.input key,
  as: :multi_value,
  input_html: {
    class: 'form-control',
    data: { 'autocomplete-url' => "/authorities/search/loc/names",
      'autocomplete' => key }
  } ,
  required: f.object.required?(key) %>
```
If you add the attribute to your view it will be active when you visit the form. 

Hyrax comes with [Questioning Authority](https://github.com/projecthydra-labs/questioning_authority) which provides some RESTful endpoints that you can use as autocomplete sources. Checkout the [QA README](https://github.com/projecthydra-labs/questioning_authority/blob/master/README.md) for a list of the authorities that it comes with (you can even make your own). 

Autocomplete in Hyrax currently uses jQuery UI Autocomplete. Hyrax stores the [jQuery UI Autocomplete options and 
source](http://jqueryui.com/autocomplete/#remote-jsonp) in ES6 classes. Unless you need to make specific query that requires you to change the options and source, the JS will use a default query from [default.es6](https://github.com/projecthydra-labs/hyrax/blob/master/app/assets/javascripts/hyrax/autocomplete/default.es6). This should just 
work with the QA vocabs. 

The autocomplete for the `Work` field requires different a different source and options so it has a different class: [work.es6](https://github.com/projecthydra-labs/hyrax/blob/master/app/assets/javascripts/hyrax/autocomplete/work.es6).
After creating your own class, you will need to import it and an additional case to the autocomplete method in [autocomplete.es6](https://github.com/projecthydra-labs/hyrax/blob/master/app/assets/javascripts/hyrax/autocomplete.es6)

### Default autocomplete fields

There are some fields that come with autocomplete turned on by default. The location field
uses GeoNames, but you need to provide credentials in `config/hyrax.rb` for it to work. 

Subject & language use local vocabularies by default. You can read about how to set those up 
in the [Questioning Authority README]((https://github.com/projecthydra-labs/questioning_authority/blob/master/README.md). 

# Activating a dropdown with authorities

To create your own select input with authorities you extend the `QASelectService` class at `/app/services/hyrax/name_authorities.rb`

```ruby
module Hyrax
  # Provide select options for the creator field
  class NameAuthorities < QaSelectService
    def initialize
      super('names')
    end
  end
end
```

You initialize the service with the name of a YAML file `config/authorities/names.yml` that contains a list of authorities:

```yaml
terms:
    - id: /authorities/search/loc/names
      term: LOC Names
      active: true
    - id: /authorities/search/assign_fast/all
      term: FAST
      active: true
```

Then in the partials for the input field at `/app/views/records/edit_fields/_creator.html.erb`:

```erb
<% name_authorities = Hyrax::NameAuthorities.new %>

<%=
  f.input key,
  as: :multi_value,
  input_html: {
    class: 'form-control',
    data: { 'autocomplete-url' => "/authorities/search/loc/names",
      'autocomplete' => key }
  } ,
  required: f.object.required?(key) %>

<%= f.input key, collection: name_authorities.select_active_options, label: false %>
```
Create a JS file in your app to activate the dropdown:

```javascript
Blacklight.onLoad( function() {
	Hyrax.authoritySelect({ selectBox : "select#work_creator", inputField : "input.work_creator" });
});
```



