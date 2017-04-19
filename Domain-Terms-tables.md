There are a few SQL tables in the Sufia app that are for handling *domain terms* (which is librarian for fields with an autocomplete). 

Below is a quick summary of these tables: 

* `domain_terms` lists the fields (terms) that are handled for each model.

* `domain_terms_local_authorities` assigns an "authority" for each of the fields listed in `domain_terms`. For example, if `generic_files.language` is a field listed in `domain_terms`, we can have two different authorities listed for it in `domain_terms_local_authorities`.

* `local_authorities` lists what authorities we have locally (e.g. lc_subjects, lexvo_languages)

* `local_authorities_entries` contains the actual data used in the autocomplete for all authorities (e.g. you'll find rows with language names and whatever terms are listed in `domain_terms`)

* `subject_local_authority_entries` is similar to `local_authorities_entries` but it is specific for subjects. See below on why this is handled separately.

Below are some notes that I took from a conversation with @mjgiarlo a few months ago when I was trying to understand this topic. 

> domain terms is basically a list of metadata form fields that you want to have autosuggest turned on for language, subject, etc.
>
> local authorities are locally harvested lists of terms. a local authority might be "lexvo", a vocabulary containing 5000+ language keywords
>
> domain_terms_local_authorities is the mapping table, where you say, I want language backed by these two local authorities, and I want subject backed by these other three authorities.
>
> This lets you put multiple authorities behind a term if you so desire local authority entries is where all the terms themselves are stored except for subjects, which is a HUGE vocabulary, so cam156 broke those out into their own entries table for performance purposes.
>
> We're only using two vocabularies now, if memory serves: library of congress subject headings (LCSH) for subject, and lexvo for language. LCSH is very widely used in the library world.  Lexvo is one of the leading ontology in the RDF ecosystem. 
>
> We have also wired our Location field to geonames' autosuggest API but since that's all via the network, it doesn't use these domain_terms/authorities tables. Geonames is pretty widely used.

Ideally these domain_terms tables should be replaced with [Questioning Authority](https://github.com/projecthydra/questioning_authority)  

