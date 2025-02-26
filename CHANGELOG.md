# Changelog

All notable changes to `FAIRshare` will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

## v1.2.0 - 2022-03-31

### Feature additions

- You will now be warned when you select a GitHub repo that doesn't have the appropriate permissions. This should prevent you from accidentally uploading to a repo that you don't have permission to.
- Added a button to let you view the repo if you decide to not publish with FAIRshare.
- Added support for reading pre-existing `codemeta` files directly from your GitHub repository.

### Bug fixes

- Fixed an issue where app log files were not being created in the correct location.
- Fixed an issue where incorrect file keys were being read from the `codemeta.json` file.
- Fixed some issues where certain GitHub repositories were not returning a license correctly. This was causing FAIRshare to assume an invalid license existed on the repository.
- Fixed an issue where an publishing a new version of a Zenodo deposition where no orcids for authors were provided could cause the app to freeze.

## v1.1.0 - 2022-03-21

### Feature additions:

- Added indicators to show when FAIRshare is updating to a new version.

### Bug fixes:

- On Linux the backend server was not being bundled correctly. This has been fixed to start on app launch.
- Fixed an issue where the splash screen on macOS was rendering a shadow.

## v1.0.0 - 2022-03-18

### Feature additions:

- Added the ability to upload your dataset to Zenodo with the appropriate metadta.
- Added support for curating GitHub repositories to Zenodo.

#### Further notes:

- First stable release of FAIRshare.
