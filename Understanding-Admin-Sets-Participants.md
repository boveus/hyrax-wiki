## Participants
### Impact on Admin Set Access

When participants are added to an Admin Set, access to the admin set will be assigned as follows.

* **Managers** - edit_access for the Admin Set - able to set admin set metadata
* **Depositors** - no access levels assigned
* **Viewers** - no access levels assigned

### Impact on ability to add works to an Admin Set

When a user is creating/editing a work, the Relationships tab has a selection list admin sets to which a user can add.  Participants that can add to an Admin Set are.

* **Managers** - can add to the admin set
* **Depositors** - can add to the admin set
* **Viewers** - can _**NOT**_ add to the admin set

### Impact on Work Access

Participant access to a work is assigned when a new work is saved.  Based on the Admin Set selected on the Relationships tab when creating a NEW work, access to the work will be assigned as follows.

* **Managers** - edit_access for the work assigned
* **Depositors** - no access levels assigned
* **Viewers** - read_access for the work assigned

Once a work is created and access is assigned, the Admin Set's participants have no more bearing on the work.  That is, if the participants are changed in the Admin Set, any existing works remain unchanged.  Any new works will get access assigned based on the new participants for the Admin Set. 

Also, if you edit a work and changed it's Admin Set, the membership of the work will be set to the new Admin Set, but the access assignments will remain unchanged.