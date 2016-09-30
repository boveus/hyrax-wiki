_This is an alpha feature_

# Introduction

Curation Concern's workflow is a database-driven implementation of a [finite state machines](https://en.wikipedia.org/wiki/Finite-state_machine). There exist other Ruby implementations of FSMs ([AASM gem](https://github.com/aasm/aasm) but these state machines are defined in code.

# Workflow Modeling

Grab a whiteboard and draw up your workflow.

* Circles represent states
* Lines represent actions
* Determine what roles are required to take an action
* Determine what roles can see an item in the given state
* When changing state, what else should happen (e.g. send notifications, update metadata)

Consider that some roles will be assigned to all items in the workflow (e.g. Metadata Reviewer) whereas some roles will be assigned to single items in the workflow (e.g. an Advisor Reviewer).

