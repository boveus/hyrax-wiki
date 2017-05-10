
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

Issues that may arise mapping  Hyrax's collection model, this new model we cannot go:
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
Intention is to create a gem that has base level behaviors that anyone who is using PCDM can use, and those that implement PCDM object can use those behaviors to ensure our relationships are consistent. Trying to stick with the coding goal to stick with one class. So a separate gem that the whole community can use and then a specific implementation of this abstract implementation for hyrax.

PCDM object has:
collection, object, and file

Hyrax level:
implement generic file and hyrax work, both of which are PCDM objects

Potential Action Items for next meeting
-Need to think about the discovery phase, because it will determine how we migrate things
-Talk more about UI and higher things next meeting.
-Should we organize into multiple teams?
-How will scrums be handled?
-How will tickets be assigned?