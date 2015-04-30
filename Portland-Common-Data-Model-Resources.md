A list of resources related to the Portland Common Data Model (PDCM) and its implementation in Sufia

* The master document on the PCDM at Duraspace: https://wiki.duraspace.org/display/FF/Portland+Common+Data+Model

* Initial diagram of the [Sufia PCDM Initial Implementation Model](https://docs.google.com/drawings/d/1uTbg0FPQDoa2zN6p1I37m-M3CFnlx85Mp9CEyRw-rf4)

* Lynette's initial wireframes: https://wiki.duraspace.org/display/hydra/Sufia+Sprint+UI+Design

* Lynette's notes on [GenericWork - GenericFile Model, Design, and Implementation Discussions](https://wiki.duraspace.org/display/FF/GenericWork+-+GenericFile+Model%2C+Design%2C+and+Implementation+Discussions)

* [List of tickets](https://github.com/projecthydra/sufia/issues?q=is%3Aopen+is%3Aissue+milestone%3A%22May+Hydra+PCDM%22) to be worked on during the Sprint in May. You can also look at them via the [Waffle board](https://waffle.io/projecthydra/sufia?milestone=May%20Hydra%20PCDM). 

* *Warning, here be dragons*. The following two documents were created during LDCX in the last week of March. These are very rough notes that people captured during the sessions but they provide some context if you are curious of how the whole thing came together: [PCDM Mappings - Reference Diagrams for Comment](https://wiki.duraspace.org/display/FF/PCDM+Mappings+-+Reference+Diagrams+for+Comment) and [Sufia move to PCDM](https://docs.google.com/document/d/1-TOtzXs87U0yOWjt34xO0dwsvIcRvIniZww5LQEwA6o/edit)

## Notes from PCDM Planning Meeting 1

D - Descriptive metadata
A - access
T - technical metadata
B - binary content stream

What is the difference in Collections, Works, Generic Files
Collections have works 
Works have generic files
Generic Files have these streams

file - individual item that someone can upload to the system
work - entity that you want to have in the system, this is where descriptive metadata will be stored.
collection - larger set of work

Issues that may arise mapping  Sufia's collection model, this new model we cannot go:
Collection -> Work -> Collection 
     -the restriction of this data model, works cannot have collections
Use Case:
     If a user wants to make another users collection part of their work, they can't.
This model allows a collection to have multiple works and a work to have multiple files


Access Questions:
-if you have read or edit access of something then you can add that to your collection, if you lose permissions then it would be gone from your collection

Will content, thumbnail, and extracted text will always be there (in the GF)?
-content: no
-extract text: try to but sometimes it is empty
-thumbnail: not always

At this point we will have empty pointers to these files if they don't exist. We may go back and update that so there are not empty pointers to files, but out of scope for current sprint

Lynette has a point of having additional streams being part of a work -- not sure of that

***Goal of the sprint:***
Intention is to create a gem that has base level behaviors that anyone who is using PCDM can use, and those that implement PCDM object can use those behaviors to ensure our relationships are consistent. Trying to stick with the coding goal to stick with one class. So a separate gem that the whole community can use and then a specific implementation of this abstract implementation for sufia.

PCDM object has:
collection, object, and file

Sufia level:
implement generic file and sufia work, both of which are PCDM objects

Potential Action Items for next meeting
-Need to think about the discovery phase, because it will determine how we migrate things
-Talk more about UI and higher things next meeting.
-Should we organize into multiple teams?
-How will scrums be handled?
-How will tickets be assigned?
