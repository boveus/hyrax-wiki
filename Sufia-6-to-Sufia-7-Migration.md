This is a work in progress to document my thoughts on how we could import Sufia 6 Generic Files into Sufia 7 Generic Works -- Hector

TL;DR. Two steps. From a Sufia 6 application export to a JSON file the metadata of all the Generic Files. From a Sufia 7 application read the JSON file and create GenericWorks/FileSet/Files. 

## Long version

### Step 1 (from a Sufia 6 application)
In a Sufia 6 application write a rake task as follows:
```
desc "Produce a JSON file with the metadata for the indicated generic files ids"
task :export => :environment do
  ids = ["1g05fb82g", "rv042t345"]
  total = ids.count
  i = 0
  open("generic_files.json", "w") do |file|
    file.puts "["
    i = 0
    ids.each do |id|
      i += 1
      puts "Processing #{i}/#{total} #{id}"
      gf = GenericFile.find(id)
      export_gf = ExportGenericFile.from_gf(gf)
      file.puts export_gf.to_json + (i < total ? "," : "")
    end
    file.puts "]"
  end
end
```
This rake task will output a JSON file with the metadata (but no the content) of the files to export.

A proof of concept for the `ExportGenericFile` class can be found at https://github.com/projecthydra/sufia/blob/migration_to_sufia7/lib/sufia/export_generic_file.rb


### Step 2 (from a Sufia 7 application)
In a Sufia 7 application write a rake task as follows:
```
desc "Imports Sufia 6 GenericFiles from ./generic_files.json into Sufia 7 GenericWorks"
task :import => :environment do
  source_file_name = "./generic_files.json"
  sufia6_user = "fedoraAdmin"
  sufia6_password = "fedoraAdmin"
  sufia6_root_uri = "http://localhost:8983/fedora/rest/dev"
  # Will be true for the real import
  # (leave as false so that we can re-run the import without running into duplicate IDs)
  preserve_ids = false

  json = File.read(source_file_name)
  generic_files = JSON.parse(json, object_class:OpenStruct)
  puts "Processing #{generic_files.count} generic_files"
  generic_files.each do |gf|
    importer = GenericWorkImport.new(sufia6_user, sufia6_password, sufia6_root_uri, preserve_ids)
    gw = importer.work_from_gf(gf)
    file_set_id = gw.file_sets[0].id
    file_id = gw.file_sets[0].files[0].id
    puts "Imported work: #{gw.id} fileset: #{file_set_id} file: #{file_id}"
  end
end
```
This take task will read the JSON produced in Step 1 and create the proper GenericWork/FileSet/File objects in the Sufia 7 application by mapping the data from the GenericFile to the correct Sufia 7 objects.

Since the content of the file is not stored in the JSON application, we read this content straight from the Fedora instance used by the Sufia 6 application at the time that we import the file to the Sufia 7 application.

A proof of concept the `GenericWorkImport` class shown in this rake task can be found at https://github.com/projecthydra/sufia/blob/migration/app/models/generic_work_import.rb
