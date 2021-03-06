Hyrax translates user interface strings into multiple languages, and we don't expect developers to manually translate all internationalization (i18n) strings in their pull requests. (English alone suffices.) To ensure we do not create gaps in Hyrax's translations over time, we use the `i18n-tasks` gem (using the [Google Translate API](https://cloud.google.com/translate/)) to create new translations and to add missing translations at release time.

# Setup

Here are the steps to get up and running with `i18n-tasks`:

* The `i18n-tasks` gem is included as a development dependency in Hyrax, and configured at `config/i18n-tasks.yml`.
* **Set up the Google Translate API key**: Follow the [instructions](https://github.com/glebm/i18n-tasks#google-translate) provided by the `i18n-tasks` gem. Do **not** add your API key to `config/i18n-tasks.yml`; store it with the rest of your shell config (e.g., `~/.bash_profile`) as a new environment variable called `GOOGLE_TRANSLATE_API_KEY`.

# Update Translations

When you're ready to create and update translations based on the current state of the English strings, run the following  commands in this order:

1. **Normalize and sort English translations**: `i18n-tasks normalize en`
2. **Add missing translations**: `i18n-tasks translate-missing --from en es zh fr it de pt-BR`
3. **Remove removed translations**: `for locale in de es zh it fr pt-BR; do i18n-tasks missing -t diff -f yaml en | i18n-tasks tree-rename-key en $locale | i18n-tasks data-remove; done`

Then add, commit, and push your changes.

# Add New Translation

Let's use Arabic as an example.

First, run `i18n-tasks translate-missing --from en ar`, and will see three new files created: 

* `config/locales/hyrax.ar.yml`
* `config/locales/simple_form.ar.yml`
* `lib/generators/hyrax/templates/config/locales/hyrax.ar.yml`

That is *most* of what you need, but not all of it. Because `i18n-tasks` will not [auto-translate yml.erb files](https://github.com/glebm/i18n-tasks/issues/253), we must handle these files manually when adding a new language. (The yml.erb files in question are named `locale.LANGUAGE.yml.erb`, located in `lib/generators/hyrax/work/templates`, and they are generated into downstream applications whenever new work types are created.)

You will need to hand-create this file. 

1. Copy `lib/generators/hyrax/work/templates/locale.en.yml.erb` to a new file at `lib/generators/hyrax/work/templates/locale.ar.yml.erb`.
2. You only need to translate one word in this new file, the word "works" (as in "works of art", not the present tense conjugation of the verb "to work"). To get the translated value of the word "works", grab the existing translation from `config/locales/hyrax.ar.yml` at the path of `ar.hyrax.admin.sidebar.works`. (Or enter the translation, if you speak the language fluently!) In this case, the translation we have is أعمال.
3. Replace the string at `ar.hyrax.select_type.<%= file_name %>.description` from `description: "<%= human_name %> works"` to `description: "<%= human_name %> أعمال"`.

Finally... add, commit, and push your changes, and *Bob ist Dein Onkel*.