If you want to see missing internationalization keys, set an environment variable called `I18N_DEBUG`, for example, in bash:

```bash
export I18N_DEBUG=true
```

And when running the test suite, output such as follows is written to `.internal_test_app/log/test.log`:

```console
[i18n-debug] en.simple_form.labels.file_set.files => nil
[i18n-debug] en.simple_form.labels.file_set.files => nil
[i18n-debug] en.simple_form.labels.defaults.files => "Upload a file"
[i18n-debug] en.simple_form.required.text => "required"
[i18n-debug] en.simple_form.required.mark => "*"
[i18n-debug] en.simple_form.required.html => "<span class=\"label label-info required-tag\">required</span>"
[i18n-debug] en.simple_form.hints.file_set.files => nil
[i18n-debug] en.simple_form.hints.file_set.files => nil
[i18n-debug] en.simple_form.hints.defaults.files => nil
[i18n-debug] en.simple_form.hints.defaults.files => nil
[i18n-debug] en.simple_form.hints.file_set.files => nil
[i18n-debug] en.simple_form.hints.file_set.files => nil
[i18n-debug] en.simple_form.hints.defaults.files => nil
[i18n-debug] en.simple_form.hints.defaults.files => nil
[i18n-debug] en.activemodel.models.file_set => nil
[i18n-debug] en.activemodel.models.active_fedora/base => nil
[i18n-debug] en.helpers.submit.file_set.update => nil
[i18n-debug] en.helpers.submit.update => "Save"
```
