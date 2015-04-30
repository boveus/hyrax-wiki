There are a few SQL tables in the Sufia app that are for handling *domain terms* (which is librarian for fields with an autocomplete). 

Below is a quick summary of these tables: 

* `domain_terms` lists the fields (terms) that are handled for each model.

* `domain_terms_local_authorities` assigns an "authority" for each of the fields listed in `domain_terms`. For example, if `generic_files.language` is a field listed in `domain_terms`, we can have two different authorities listed for it in `domain_terms_local_authorities`.

* `local_authorities` lists what authorities we have locally (e.g. lc_subjects, lexvo_languages)

* `local_authorities_entries` contains the actual data used in the autocomplete for all authorities` (e.g. you'll find rows with language names and whatever terms are listed in `domain_terms`)

* `subject_local_authority_entries` is similar to `local_authorities_entries` but it is specific for subjects. Not sure why this is handled separately.

 