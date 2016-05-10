This is a work in progress to document my thoughts on how we could import Sufia 6 Generic Files into Sufia 7 Generic Works -- Hector

TL;DR. Two steps. From a Sufia 6 application export to a set of JSON files the metadata of all the Generic Files. From a Sufia 7 application read the JSON files and create GenericWorks/FileSet/Files. 

## Long version

### Step 1 (from a Sufia 6 application)
In a Sufia 6 application export the data of the Generic Files and Collections to JSON files. Branch [export_s6 in ScholarSphere](https://github.com/psu-stewardship/scholarsphere/tree/export_s6) has a proof of concept of how this could be implemented. The code to handle the export is under `lib/export` folder. File `lib/export/service.rb` is the main driver that calls the other programs in this folder to export individual Generic Files and Collections. A basic set of `rake` tasks to calls this service can be found in `lib/tasks/export.rake`. Below is an example of how these tasks could be used:

```
# export all the GenericFile IDs to a file
rake gf_ids > gf_ids.txt

# export the metadata of the GenericFiles to JSON files
rake export_gf[gf_ids.txt]
```

Rake task `export` will output to a JSON file the metadata for each file, but not the actual binary of the file. The binary will be read from Fedora at the time of the import (more on this later.)


### Step 2 (from a Sufia 7 application)
In a Sufia 7 application write a rake task as follows:
```
require "../lib/sufia/import/import_service.rb"

desc "Imports Sufia 6 GenericFiles into Sufia 7 GenericWorks"
task :import => :environment do
  # Credentials for the Fedora instance with the content of the files to be imported
  # (this is the Fedora instance used by the *Sufia 6* application)
  user = "fedoraAdmin"
  password = "fedoraAdmin"
  root_uri = "http://localhost:8983/fedora/rest/dev"

  # Will be true for the real import
  # (leave as false so that we can re-run the import without running into duplicate IDs)
  preserve_ids = false

  # Files exported from Sufia 6
  files_to_import = File.join(Dir.pwd, "gf_*.json")
  service = Importer::ImportService.new(user, password, root_uri, preserve_ids)
  service.import_files(files_to_import)
end
```
This take task will read the JSON files produced in Step 1 and create the proper `GenericWork/FileSet/File` objects in the Sufia 7 application by mapping the data from the `GenericFile` to the correct Sufia 7 objects.

Since the content of the file is not stored in the JSON files, we read this content straight from the Fedora instance used by the Sufia 6 application at the time that we import the file to the Sufia 7 application.

A proof of concept the `ImportService` class shown in this rake task can be found at https://github.com/projecthydra/sufia/blob/migration_from_sufia6/lib/sufia/import/import_service.rb 

This code will be merged into the Sufia 7 master branch.


## Things not yet covered
* Multiple versions of files
* Some fields are still missing 
* Objects still not accounted for (collections, batches, rights, et cetera)
* This proof of concept only imports the original content of the Sufia 6 GenericFile and it let's Sufia 7 regenerate the derivatives (e.g. thumbnail, fulltext) and re-characterize the file.
* Error handling