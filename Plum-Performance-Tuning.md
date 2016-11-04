## Introduction
These are some notes on performance tuning [Plum](https://github.com/pulibrary/plum/).  We hope 
to fill these out more, but hope this is useful to help other people working on scaling up and 
addressing performance issues.

## Upgrading Fedora
We were seeing scalability problems with Fedora 4.5 and tried using the Postgresql database backend
instead of LevelDB (both for performance and because of corruption issues with LevelDB).  We found
Fedora scaled better with Postgresql than LevelDB, but was slower.  We then upgraded to the
Modeshape5 branch (which has now been released as part of Fedora 4.7), and found it performed as
fast as LevelDB, but with the scalability and reliability benefits of Postgresql.

## Disabling Tesseract
Most of our works are books, and so we have a background job to run Tesseract to OCR the page
images to support fulltext search.  We immediately noticed that Tesseract was a CPU hog and slowed
down all of the ingest workers.  We initially set it to the lowest priority, but eventually
disabled it completely to allow ingest to proceed at a decent pace.  We plan to perform OCR after
ingest.

## Graph Copying
Updating our largest objects was taking more than 8 minutes.  Profiling showed that most of the time was being spent copying RDF graphs unnecessarily.  A [deceptively simple update](https://github.com/projecthydra/active_fedora/commit/f6ac3fbe04d5c0fedb64b7c04a34ca207d29f7f7#diff-4827a9cc4420720ae18bf842f60f18f5R46) 
avoided the vast majority of graph copying, and reduced save times dramatically.

## Reversing Collection Membership
We saw a lot of slowdown when trying to ingest works in large collections.  As the collection grew,
ingesting each works got slower and slower, because the time to update and save the collection was
growing with the number of membership links.  To address this, we reversed the collection
membership relationship in Curation Concerns (see [CC#901](https://github.com/projecthydra/curation_concerns/pull/901),
having works link to collections.  So instead of large collections having many links to their
member works, each work links to a single collection.  This dramatically improved work ingest times.

## Ordering After Ingesting Files
Our books have anywhere from a handful of pages up to over 1200.  We noticed that ingesting each
page got slower and slower over the course of ingesting a book, for much the same reason as the
collection membership issue above.  To address this, we updated our
[ingest job](https://github.com/pulibrary/plum/blob/master/app/jobs/ingest_mets_job.rb) to ingest
all the files first, and then apply the ordering to organize them.  This allowed ingesting files to
proceed a contant rate and not slow down over the course of larger books.

## Disabling Tesseract, Again
We noticed that FITS was taking a very long time: ingesting files and generating JPEG 2000
derivatives was taking roughly 10-15 seconds each, but running FITS was taking 90-120 seconds.
After looking closely at what processes were running, we noticed that FITS (which bundles several
tools) was running Apache Tika, which in turn was running Tesseract.  We [disabled Tika in our FITS
configuration](https://github.com/ucsdlib/ansible-role-fits/pull/2), and this reduced the time to
run FITS down to 15-20 seconds per file.

## Beefing Up Fedora
Not surprisingly, we found that Fedora was a significant factor limiting ingest speed.  Our Fedora
VM had 1 CPU and 10GB or RAM.  We increased that to 4 CPUs and 20 GB of RAM, and found our ingest
jobs were performing much better and the CPU load on the Fedora machine was much lower.

## Multiple Workers
We were running our background jobs on a separate VM, which was showing a pretty high load average
(around 4.0).  But Fedora had a pretty low load average, so we added a scond worker VM, and it
bascially doubled the ingest rate.

## Database Minter State to Support Multiple Workers
After we added a second worker VM, we started seeing a lot of Fedora errors (404 errors on
resources that should exist, long sequences of HEAD/GET requests on existing objects, etc.).  And
when creating objects, we would sometimes see a very long lag as it tried to find a PID that wasn't
in use already.  We realized that our Hydra head and the two workers all had separate PID minter
statefiles, and so were generating duplicate PIDs.  So we switched to [storing our PID minter state
in the database](https://github.com/projecthydra/active_fedora-noid#use-database-based-minter-state).
We had some trouble getting this to work, and eventually got it working by [manually deleting an
invalid minter state](https://gist.github.com/escowles/c5f1b505954ab368fa659614a2a85610).