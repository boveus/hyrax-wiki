## Are you migrating from 2.0.x to 2.1.0?  

Add your migration experiences to this doc to help out your fellow migrators.

---

### Change to PermissionTemplates

PermissionTemplates had a database migration changing column named `admin_set_id` to `source_id`.  If you have code customizations for PermissionTemplates or have tests that create PermissionTemplates, you will need to rename `admin_set_id` to `source_id`.

---

### Error on overridden Home page

**This issue was encountered on release candidates, but fixed prior to the final Hyrax 2.1.0 release.**

View Home page...

* FAILURE: `undefined local variable or method 'create_work_presenter' in file /app/views/_masthead.html.erb where line #1`
* SOLUTION:  Verified that the `render /shared/select_work_type_modal` was in stable-2.0 but is not in hyrax 2.1.0.  So this was not our customization.  I removed that line from our copy of `_masthead.html.erb`

---

### Error on overridden `views/hyrax/my/works/index.html.erb`

**This issue was encountered on release candidates, but fixed prior to the final Hyrax 2.1.0 release.**

* FAILURE: Encountered when running our spec tests: `undefined local variable or method 'create_work_presenter'` ... `Did you mean?  @create_work_presenter`
* SOLUTION: Change references from `create_work_presenter` in overridden `views/hyrax/my/works/index.html.erb` to `@create_work_presenter` 

---

### Fix performance issue caused by solr-suggest after 200 objects are in the repository

**Recommended for new and migrating apps**

#### PROBLEM
solr-suggest degrades performance when adding collections and works to the repository starting after 200 objects have been added.  See the graphs of performance below.

![image](https://user-images.githubusercontent.com/6855473/40363719-6096858e-5d9e-11e8-9a89-287ccdb44430.png)

#### SOLUTION
Reference:  [samvera/active_fedora#1311](https://github.com/samvera/active_fedora/pull/1311)

In the future, Hyrax will install with solr-suggest turned off by default.  For now you need to modify the following to turn it off.

* make the same changes to both /solr/conf/solrconfig.xml AND /solr/config/solrconfig.xml
* comment out the following lines

```
  <!-- TURN OFF SUGGEST - it causes performance issues -->
  <!--
  <searchComponent name="suggest" class="solr.SuggestComponent">
    <lst name="suggester">
      <str name="name">mySuggester</str>
      <str name="lookupImpl">FuzzyLookupFactory</str>
      <str name="suggestAnalyzerFieldType">textSuggest</str>
      <str name="buildOnCommit">true</str>
      <str name="field">suggest</str>
    </lst>
  </searchComponent>

  <requestHandler name="/suggest" class="solr.SearchHandler" startup="lazy">
    <lst name="defaults">
      <str name="suggest">true</str>
      <str name="suggest.count">5</str>
      <str name="suggest.dictionary">mySuggester</str>
    </lst>
    <arr name="components">
      <str>suggest</str>
    </arr>
  </requestHandler>
  -->
```