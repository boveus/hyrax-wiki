This page parallels the guide to [Customizing Metadata](https://github.com/projecthydra/sufia/wiki/Customizing-Metadata) but attempts to deal with complex RDF metadata, where a nested sub-form will appear when editing.

> This page is probably going about things completely the wrong way. If anyone can correct it and fill in the
> gaps to produce a working demonstration which can be persisted in the repository and reloaded that would be
> wonderful.

Table of Contents
=================

  * [Override the GenericFile model](#override-the-genericfile-model)
    * [Add the property](#add-the-property)
    * [Try the property at the Rails console](#try-the-property-at-the-rails-console)
  * [Extend the presenter](#extend-the-presenter)
    * [Set the controller to use our presenter](#set-the-controller-to-use-our-presenter)
  * [Create edit forms](#create-edit-forms)
    * [Set the controllers to use our forms](#set-the-controllers-to-use-our-forms)
  * [Labels and help text](#labels-and-help-text)
  * [Required/non-required fields](#requirednon-required-fields)
  * [Field repeatability](#field-repeatability)

# Override the GenericFile model

As for the basic case, we override Sufia's GenericFile class to add the complex metadata to the model by defining it in our app.

```ruby
# app/models/generic_file.rb
class GenericFile < ActiveFedora::Base
  include Sufia::GenericFile
end
```

## Add the property

We add the property declaration into the `GenericFile` class, and define a further class to declare the properties within. In this example we are using non-standard predicate names with made-up URIs, just as an example. You would probably want to define the vocabulary properly elsewhere.
`TODO`: define how the fields are to be indexed in Solr.

```ruby
  property :dimensions, predicate: ::RDF::URI('http://example.org/terms/dimensions'), class_name: "GenericFile::Dimensions"

  accepts_nested_attributes_for :dimensions

  class Dimensions < ActiveTriples::Resource
    configure type: ::RDF::URI('http://example.org/terms/dimensionSet')
    property :height, predicate: ::RDF::URI('http://example.org/terms/height')
    property :width, predicate: ::RDF::URI('http://example.org/terms/width')
    property :depth, predicate: ::RDF::URI('http://example.org/terms/depth')
  end
```

When you're done the file should look like this:
```ruby
# app/models/generic_file.rb
class GenericFile < ActiveFedora::Base
  include Sufia::GenericFile

  property :dimensions, predicate: ::RDF::URI('http://example.org/terms/dimensions'), class_name: "GenericFile::Dimensions"

  accepts_nested_attributes_for :dimensions

  class Dimensions < ActiveTriples::Resource
    configure type: ::RDF::URI('http://example.org/terms/dimensionSet')
    property :height, predicate: ::RDF::URI('http://example.org/terms/height')
    property :width, predicate: ::RDF::URI('http://example.org/terms/width')
    property :depth, predicate: ::RDF::URI('http://example.org/terms/depth')
  end
end
```

## Try the property at the Rails console

Now we can try defining an object with dimensions at the Rails console to see if it works. I've shown the expected output after each command.
```
$ rails c
Loading development environment (Rails 4.1.8)
irb(main):001:0> f = GenericFile.new(id: 'Test-1', title: ['Title of resource'])
  LocalAuthority Load (1.2ms)  SELECT  "local_authorities".* FROM "local_authorities"  WHERE "local_authorities"."name" = 'lc_subjects' LIMIT 1
  LocalAuthority Load (1.4ms)  SELECT  "local_authorities".* FROM "local_authorities"  WHERE "local_authorities"."name" = 'lexvo_languages' LIMIT 1
  LocalAuthority Load (1.3ms)  SELECT  "local_authorities".* FROM "local_authorities"  WHERE "local_authorities"."name" = 'lc_genres' LIMIT 1
ActiveFedora: loading fedora config from /opt/sufia/config/fedora.yml
ActiveFedora: loading solr config from /opt/sufia/config/solr.yml
Attempted to init base path `dev`, but it already exists
=> #<GenericFile id: "Test-1", mime_type: "", format_label: [""], file_size: [""], last_modified: [""], filename: [""], original_checksum: [""], rights_basis: [], copyright_basis: [], copyright_note: [], well_formed: [], valid: [""], status_message: [], file_title: [""], file_author: [""], page_count: [""], file_language: [], word_count: [], character_count: [], paragraph_count: [], line_count: [], table_count: [], graphics_count: [], byte_order: [], compression: [], color_space: [], profile_name: [], profile_version: [], orientation: [], color_map: [], image_producer: [], capture_device: [], scanning_software: [], exif_version: [], gps_timestamp: [], latitude: [], longitude: [], character_set: [], markup_basis: [], markup_language: [], bit_depth: [], channels: [], data_format: [], offset: [], frame_rate: [], label: nil, depositor: nil, relative_path: nil, import_url: nil, part_of: [], resource_type: [], title: ["Title of resource"], creator: [], contributor: [], description: [], tag: [], rights: [], publisher: [], date_created: [], date_uploaded: nil, date_modified: nil, subject: [], language: [], identifier: [], based_near: [], related_url: [], bibliographic_citation: [], source: [], proxy_depositor: nil, on_behalf_of: nil, dimensions: [], batch_id: nil>
irb(main):002:0> f.dimensions_attributes = [{height:"10cm", width:"30cm"}, {width:"25cm", depth:"100mm"}]
=> [{:height=>"10cm", :width=>"30cm"}, {:width=>"25cm", :depth=>"100mm"}]
irb(main):003:0> puts f.resource.dump(:ttl)

<http://127.0.0.1:8983/fedora/rest/dev/Te/st/-1/Test-1> <http://purl.org/dc/terms/title> "Title of resource";
   <http://example.org/terms/dimensions> [
     a <http://example.org/terms/dimensionSet>;
     <http://example.org/terms/height> "10cm";
     <http://example.org/terms/width> "30cm"
   ],  [
     a <http://example.org/terms/dimensionSet>;
     <http://example.org/terms/depth> "100mm";
     <http://example.org/terms/width> "25cm"
   ];
   <info:fedora/fedora-system:def/model#hasModel> "GenericFile" .
=> nil
irb(main):004:0>
```

# Extend the presenter

## Set the controller to use our presenter

# Create edit forms

## Set the controllers to use our forms

# Labels and help text

`TODO`

# Required/non-required fields

`TODO`

# Field repeatability

`TODO`