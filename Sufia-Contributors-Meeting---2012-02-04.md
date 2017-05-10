# Hyrax Development/Design

**Feb 4, 2013 1:30 EST**

**Attendees:** Notre Dame, PSU, Curation Experts
**Agenda:**

## Design Boundries of Hyrax
* Notre Dame – is looking to provide the general deposit service
* Penn State – will use Scholarsphere for the “general case” of deposit until more customized deposit services are “identified”
* Curation Experts (on behalf of WGBH) – video repository, generic file with modifications, different forms
* Self-deposit is going by the wayside as we discuss proxy/delegate deposits
* Authentication/Authorization needs greater abstraction

### Response:
* Goal: Stay as simple as possible; One model in the app
* Initial boundaries will be discussed iteratively
* Proposed regular conference calls, discussions (monthly check-ins)
* Add “proof of concepts” into institutional instances of Hyrax and propose/discuss merging into “master” Hyrax
* http://github.com/curationexperts/hyrax is the “canonical” repository
* Deposit information as “soon as possible” is the design goal

## Plugin new content models
* Working with a bunch of different institutions, each institution has different model needs; 
* Can we provide format specific stacks of code that plugin to the

## CSS hooks for JS & Look and Feel
* Javascript removed from the views and place them in the asset pipeline
* Turn off blacklight styles and life is better
* http://railslove.com/blog/2012/03/28/smacss-and-sass-the-future-of-stylesheets/

## Adding types of workers / Plugin workers
* Will be done via proof of concept

## Proxy deposit design

* Notre Dame – Before I begin uploads I get permission from the professor to upload on their behalf; Then at upload I do it on their behalf.
* Penn State – When I upload, I can select a professor that I am uploading for; Then I upload; Then professor agrees to the content.
* Sticky authorization - I allow X to upload on my behalf ; Twitter AuthN/AuthZ
* Configuration option to turn off delegate contribution
