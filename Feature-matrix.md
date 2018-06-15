| Feature | How to enable | Since: |
| ------- | ------------- | ------------------- |
| Multiple file upload | automatically enabled | 1.0.0
| Recursive folder upload | automatically enabled (requires Chrome browser) | 1.0.0
| Flexible user- and group-based access controls | automatically enabled | 1.0.0
| Generation and validation of identifiers | automatically enabled | 1.0.0
| Generation of derivatives | automatically enabled | 1.0.0
| Fixity checking (using Fedora's fixity service) | automatically enabled | 1.0.0
| Version control | automatically enabled | 1.0.0
| Characterization of uploaded files | automatically enabled | 1.0.0
| Forms for batch editing metadata | automatically enabled | 1.0.0
| Faceted search and browse | automatically enabled | 1.0.0
| Social media interaction | automatically enabled | 1.0.0
| User profiles | automatically enabled | 1.0.0
| User dashboard for file management | automatically enabled | 1.0.0
| Highlighted files on profile | automatically enabled | 1.0.0
| Sharing w/ groups and users | automatically enabled | 1.0.0
| User notifications | automatically enabled | 1.0.0
| Activity streams | automatically enabled | 1.0.0
| Single-use links | automatically enabled | 1.0.0
| Google Scholar-specific metadata embedding | automatically enabled | 1.0.0
| Schema.org microdata, Opengraph/Twitter cards | automatically enabled | 1.0.0
| User-managed collections for grouping files | automatically enabled | 1.0.0
| Full-text indexing & searching | automatically enabled | 1.0.0
| Responsive, fluid, Bootstrap 3-based UI | automatically enabled | 1.0.0
| Featured works and researchers on homepage | automatically enabled | 1.0.0
| Proxy deposit and transfers of ownership | automatically enabled | 1.0.0
| Questioning Authority | automatically enabled | 1.0.0
| ResourceSync capability lists and resource lists | automatically enabled | 1.0.0
| Contact form | automatically enabled | 1.0.0
| Administrative dashboard, w/ feature flippers | automatically enabled (for administrative users) | 1.0.0
| Flexible object model | automatically enabled: to allow zero-file works, enable in Hyrax initializer | 1.0.0
| Transcoding of audio and video files | off by default: enable in Hyrax initializer (requires `ffmpeg`) | 1.0.0
| Administrative sets (curated collections) | automatically enabled | 1.0.0
| Customizable banner image | specify a banner image in Hyrax initializer | 1.0.0
| Capture usage statistics | requires [Google Analytics ID to be specified in Hyrax initializer](https://github.com/samvera/hyrax/wiki/Hyrax-Management-Guide#capturing-usage) | 1.0.0
| Geonames integration for location-oriented metadata | requires [configuration](https://github.com/samvera/hyrax/wiki/Hyrax-Management-Guide#geonames) | 1.0.0
| Virus detection for uploaded files | install `clamav` package and follow [the management guide instructions](https://github.com/samvera/hyrax/wiki/Hyrax-Management-Guide#virus-checking) | 1.0.0
| Display usage statistics in the UI | requires [configuration](https://github.com/samvera/hyrax/wiki/Hyrax-Management-Guide#displaying-usage-in-the-ui) | 1.0.0
| Administrative users | requires [configuration](https://github.com/samvera/hyrax/wiki/Making-Admin-Users-in-Hyrax) | 1.0.0
| Citation formatting suggestions | requires configuration: enable `config.citations` in Hyrax initializer | 1.0.0
| Integration with Zotero | requires [configuration](https://github.com/samvera/hyrax/wiki/Hyrax-Management-Guide#zotero-integration) | 1.0.0
| Integration w/ cloud storage providers | requires [configuration](https://github.com/samvera/hyrax/wiki/Hyrax-Management-Guide#integration-with-dropbox-box-etc) | 1.0.0
| Background jobs | requires configuration: though jobs will automatically run via the default in-memory adapter, we recommend using an `ActiveJob` adapter like `Sidekiq` in production environments | 1.0.0
| [IIIF](http://iiif.io/) manifests | automatically enabled | 1.0.0
| [IIIF](http://iiif.io/) image server | follow [the management guide instructions](https://github.com/samvera/hyrax/wiki/Hyrax-Management-Guide#image-server) | 1.0.0
| UniversalViewer on work show page | automatically enabled for image-like assets when using IIIF image server | 1.0.0
