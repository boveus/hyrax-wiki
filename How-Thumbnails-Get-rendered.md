Thumbnails get rendered on the UI by displaying an image with the path defined in a field in Solr.  By default that field is 'thumbnail_path_ss' defined [here](https://github.com/projecthydra-labs/hyrax/blob/master/app/services/hyrax/indexes_thumbnails.rb).  Once the path is indexed into solr and object would need to be re-indexed for the thumbnail to change.

The path gets indexed into solr by the Indexer.  The default indexers can be found [here](https://github.com/projecthydra-labs/hyrax/tree/master/app/indexers/hyrax).
Each indexer uses a ThumbnailPathServer to generate the string used in solr Sound [here](https://github.com/projecthydra-labs/hyrax/blob/master/app/services/hyrax/thumbnail_path_service.rb).

You can change the default behavior by creating your own ThumbnailPathService and Indexer that includes a line to point to  you new ThumnbnailPathService

```ruby
module MyNamespace
  class MyIndexer < Hyrax::WorkIndexer
    self.thumbnail_path_service = MyNamespace::MyThumbnailPathService
  end
end
```

You will then need your work to point at your new indexer

```ruby
class MyWork
  self.indexer = MyNamespace::MyIndexer
end
```
