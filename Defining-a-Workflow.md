_This is an alpha feature_

# Introduction

Curation Concern's workflow is a database-driven implementation of a [finite state machines](https://en.wikipedia.org/wiki/Finite-state_machine). There exist other Ruby implementations of FSMs ([AASM gem](https://github.com/aasm/aasm) but these state machines are defined in code).

# Workflow Modeling

Grab a whiteboard and draw up your workflow.

* Circles represent states
* Lines represent actions
* Determine what roles are required to take an action
* Determine what roles can see an item in the given state
* When changing state, what else should happen (e.g. send notifications, update metadata)

Consider that some roles will be assigned to all items in the workflow (e.g. Metadata Reviewer) whereas some roles will be assigned to single items in the workflow (e.g. an Advisor Reviewer).

Below is one example:

![example-workflow](https://cloud.githubusercontent.com/assets/2130/19000926/1eab97e4-8713-11e6-9edc-0599fedca795.png)

# Defining a Workflow in CurationConcerns

CurationConcerns will create a default workflow configuration when you call the `rails generator curation_concerns:work`. **Before you load the default workflow into the database** via the `rake curation_concerns:workflow:load` command, make sure to review your workflow.

Below is the JSON that models the above workflow.

```json
{
  "work_types": [
    {
      "name": "example",
         "actions": [{
             "name": "submit_for_review", "transition_to": "under_review",
             "from_states": [{"names": ["new", "changes_required"], "roles": ["creating_deposit"]}]
         }, {
             "name": "request_changes", "transition_to": "changes_required",
             "from_states": [{"names": ["under_review"], "roles": ["approving_work"]}]
        }, {
             "name": "approve", "transition_to": "approved",
             "from_states": [{"names": ["under_review"], "roles": ["approving_work"]}]
        }]
     }
  ]
}
```

# Loading a Workflow

## Default Workflow

When a Curation Concerns app is created by running `rails generate curation_concerns:install`, a default workflow is created in [config/workflows/default_workflow.json](https://github.com/projecthydra/curation_concerns/blob/master/lib/generators/curation_concerns/templates/workflow.json.erb).

## Additional Workflows

You can define additional workflows following the syntax described [Defining a Workflow in CurationConcerns](#defining-a-workflow-in-curationconcerns) above.  Add the json files to `config/workflows` directory in your app.

## Loading workflows

All workflows defined in `config/workflows` directory can be loaded using the following command.

```
rake curation_concerns:workflow:load
```

This will load the workflow and create user roles.  You will want to assign users/groups to the roles.  You will be able to select the workflow for use in an admin_set.
