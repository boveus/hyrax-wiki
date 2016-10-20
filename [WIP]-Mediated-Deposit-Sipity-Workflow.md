# Why mediated deposit?

In the hydra community, there are a lot of folks who would like to be able to ingest an item and make sure it doesn't go live until the item and its metadata have been thoroughly reviewed. This is where mediated deposit comes in. In Sufia, a user can now describe groups of steps they want to take to have an item "properly" reviewed. Sufia uses what is called "workflows" to describe these groups of steps to review an item that was ingested. 

# What is a workflow?

In Sufia, a workflow is a state machine generated pragmatically by utilizing work done in a gem called Sipity (https://github.com/ndlib/sipity). Each workflow can look different depending on who has designed it. By taking this info into account, it is now possible to generate any workflow a user would like by simply describing it in a JSON format.

# Initial workflow setup

      ## Creating a workflow

      The first step in introducing a workflow in Sufia is to draw out a state machine. Pull up a whiteboard or piece of paper and diagram out what you think a workflow should look like. Here are a few tips about drawing out a state machine:

        * Draw circles or squares to describe a step in that workflow. (metadata review step, digitization review step, etc.)
        * Draw lines in between these circles or squares to describe how to transition from one state to another.
        * Determine the roles that are required by the user who would be taking an action.
        * Determine which roles can see items throughout the workflow.
        * When a state change occurs, what other actions could/should be taken?

     Here is a quick example of a state machine that describes a workflow:

![example-workflow](https://cloud.githubusercontent.com/assets/2130/19000926/1eab97e4-8713-11e6-9edc-0599fedca795.png)