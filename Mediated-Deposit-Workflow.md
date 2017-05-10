# Why mediated deposit?

In the hydra community, there are a lot of folks who would like to be able to ingest an item and make sure it doesn't go live until the item and its metadata have been thoroughly reviewed. This is where mediated deposit comes in. In Hyrax, a user can now describe groups of steps they want to take to have an item "properly" reviewed. Hyrax uses what is called "workflows" to describe these groups of steps to review an item that was ingested. 

# What is a workflow?

In Hyrax, a workflow is a state machine generated pragmatically by utilizing work done in a gem called Sipity (https://github.com/ndlib/sipity). Each workflow can look different depending on who has designed it. By taking this info into account, it is now possible to generate any workflow a user would like by simply describing it in a JSON format.

# Initial workflow setup

## Creating a workflow

The first step in introducing a workflow in Hyrax is to draw out a state machine. Pull up a whiteboard or piece of paper and diagram out what you think a workflow should look like. Here are a few tips about drawing out a state machine:

* Draw circles or squares to describe a step in that workflow. (metadata review step, digitization review step, etc.)
* Draw lines in between these circles or squares to describe how to transition from one state to another.
* Determine the roles that are required by the user who would be taking an action.
* Determine which roles can see items throughout the workflow.
* When a state change occurs, what other actions could/should be taken?

Here is a quick example of a state machine that describes a workflow:

![example-workflow](https://cloud.githubusercontent.com/assets/2130/19000926/1eab97e4-8713-11e6-9edc-0599fedca795.png)

## Translating a workflow into JSON

Now that a workflow has been conceptually figured out, it is now time to generate a JSON expression of said workflow. This JSON file, should live in `APP_ROOT/config/workflows/*.json` Sipity follows a very stringent format for this JSON expression of a state machine. 

The first expression in the JSON is a "work_types" tag. This tells the program that a list of work types are coming up. These work types need to exist in Hyrax in order for the program to fully generate the workflow needed. To generate a work type, a user can run `rails generator curation_concerns:work <NameOfWork>` in the command line. Note: This generator command will generate a default workflow, but it may not be the workflow you want.

```
Example: 
A user whom wants to generate a GenericWork work type would run
rails generator curation_concerns:work GenericWork.
This will generate a generic_work_workflow.json file.
It will serve as your workflow JSON file
```

Next comes the "actions" tag which is a list of actions that can be taken in the workflow. This is expressed in the JSON file as an array of action names with other supporting information that describe the way to get from one state to the next, what state comes next, what actions can be taken during a state change, and the roles needed to make the state change. 

Within each action is a "name" tag, a "transition_to" tag, a "from_states", and a "roles" tag. This describes the name of the action, where the state transitions to after an action taken, what state is prior to the current state, and the roles needed for making the change from the current state to the next. Below is a sample JSON file, which replicates the aforementioned state machine.

```
{
  "workflow": [
    {
      "name": "example",
         "actions": [{
           "name": "approve", "transition_to": "approved",
           "from_states": [{"names": ["under_review"], "roles": ["approving_work"]}]
         }, {
             "name": "submit_for_review", "transition_to": "approved",
             "from_states": [{"names": ["new", "changes_required"], "roles": ["creating_deposit"]}]
         }, {
             "name": "request_changes", "transition_to": "changes_required",
             "from_states": [{"names": ["under_review"], "roles": ["approving_work"]}]
        }]
     }
  ]
}
```

## How do I get this workflow into the system?

Thats a good question. Its quite simple! All you need to do now is run a simple command in the command line.
```
rails curation_concerns:workflow:load
```
and voila! Your workflow is now loaded into the system and is ready to be used. If you dont get yelled at by this rake task then you know that it worked. The verification of your workflow JSON file is very particular and will come back with errors if there is anything wrong in your workflow JSON file.

## Don't users need roles in order to process items within a workflow?

This is true. Luckily, there is a UI element that now exists in which an administrative user can add a role to a user within a particular workflow. This can be seen at `/admin/workflow_roles/?`. For development purposes, if you would like to add all users to all roles, you can do that with the following command:
```
CurationConcerns::Workflow::PermissionGenerator.call(roles: Sipity::Role.all,        
                                                     workflow: Sipity::Workflow.last,        
                                                     agents: User.all)
```
