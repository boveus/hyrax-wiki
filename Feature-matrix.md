| Feature | How to enable |
| ------- | ------------- |
| Multiple file upload | automatically enabled |
| Recursive folder upload | automatically enabled (requires Chrome browser)
| Flexible user- and group-based access controls | automatically enabled |
| Generation and validation of identifiers | automatically enabled |
| Generation of derivatives | automatically enabled |
| Fixity checking (using Fedora's fixity service) | automatically enabled |
| Version control | automatically enabled |
| Characterization of uploaded files | automatically enabled |
| Forms for batch editing metadata | automatically enabled |
| Faceted search and browse | automatically enabled |
| Social media interaction | automatically enabled |
| User profiles | automatically enabled |
| User dashboard for file management | automatically enabled |
| Highlighted files on profile | automatically enabled |
| Sharing w/ groups and users | automatically enabled |
| User notifications | automatically enabled |
| Activity streams | automatically enabled |
| Single-use links | automatically enabled |
| Google Scholar-specific metadata embedding | automatically enabled |
| Schema.org microdata, Opengraph/Twitter cards | automatically enabled |
| User-managed collections for grouping files | automatically enabled |
| Full-text indexing & searching | automatically enabled |
| Responsive, fluid, Bootstrap 3-based UI | automatically enabled |
| Featured works and researchers on homepage | automatically enabled |
| Proxy deposit and transfers of ownership | automatically enabled |
| Questioning Authority | automatically enabled |
| ResourceSync capability lists and resource lists | automatically enabled |
| Contact form | automatically enabled |
| Administrative dashboard, w/ feature flippers | automatically enabled (for administrative users) |
| Flexible object model | automatically enabled: to allow zero-file works, enable in Hyrax initializer |
| Transcoding of audio and video files | off by default: enable in Hyrax initializer (requires `ffmpeg`) |
| Administrative sets (curated collections) | off by default: enable by flipping on in administrative UI |
| Customizable banner image | specify a banner image in Hyrax initializer |
| Capture usage statistics | requires [Google Analytics ID to be specified in Hyrax initializer](https://github.com/projecthydra-labs/hyrax/wiki/Hyrax-Management-Guide#capturing-usage) |
| Geonames integration for location-oriented metadata | requires [configuration](requires [configuration](https://github.com/projecthydra-labs/hyrax/wiki/Hyrax-Management-Guide#geonames)) |
| Virus detection for uploaded files | install `clamav` package and follow [the instructions in Hyrax](https://github.com/projecthydra/curation_concerns#virus-detection) |
| Display usage statistics in the UI | requires [configuration](https://github.com/projecthydra-labs/hyrax/wiki/Hyrax-Management-Guide#displaying-usage-in-the-ui) |
| Administrative users | requires [configuration](https://github.com/projecthydra-labs/hyrax/wiki/Making-Admin-Users-in-Hyrax) |
| Citation formatting suggestions | requires configuration: enable `config.citations` in Hyrax initializer, then run `rails g hyrax:citation_config` |
| Integration with Zotero | requires [configuration](https://github.com/projecthydra-labs/hyrax/wiki/Hyrax-Management-Guide#zotero-integration) |
| Integration w/ cloud storage providers | requires [configuration](https://github.com/projecthydra-labs/hyrax/wiki/Hyrax-Management-Guide#integration-with-dropbox-box-etc) |
| Background jobs | requires configuration: though jobs will automatically run via the default in-memory adapter, we recommend using an `ActiveJob` adapter like `Sidekiq` in production environments |
