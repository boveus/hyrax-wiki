## Are you migrating from 2.0.x to 2.1.0?  

Add your migration experiences to this doc to help out your fellow migrators.

---

### Change to PermissionTemplates

PermissionTemplates had a database migration changing column named `admin_set_id` to `source_id`.  If you have code customizations for PermissionTemplates or have tests that create PermissionTemplates, you will need to rename `admin_set_id` to `source_id`.

---

### Error on overridden Home page

View Home page...

* FAILURE: `undefined local variable or method `create_work_presenter' in file /app/views/_masthead.html.erb where line #1`
* SOLUTION:  Verified that the `render /shared/select_work_type_modal` was in stable-2.0 but is not in hyrax 2.1.0.  So this was not our customization.  I removed that line from our copy of `_masthead.html.erb`

---
