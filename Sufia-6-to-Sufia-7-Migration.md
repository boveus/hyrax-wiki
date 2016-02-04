This is a work in progress and mostly documenting my thoughts on how we could import Sufia 6 Generic Files into Sufia 7 Generic Works -- Hector

TL;DR. Two steps. One: from a Sufia 6 application export to a JSON file the metadata of all the Generic Files. Two: From a Sufia 7 application read the JSON file and create GenericWorks/FileSet/Files. The content of the GenericFiles is read straight from the Fedora instance used by the Sufia 6 application (i.e. the content is not in the JSON file.)

## Long version
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
A proof of concept for the `ExportGenericFile` class can be found at https://github.com/projecthydra/sufia/blob/migration_to_sufia7/lib/sufia/export_generic_file.rb

This rake task will output a JSON file with the metadata (but no the content) of the files to export.



