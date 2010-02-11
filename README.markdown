# dsym-archiver README

> As written, this script is for iPhone OS only. Modifications are required for
> Mac OS X support (mostly removing the adhoc packaging) and to fix the
> SDK_NAME trigger.

This script is intended to be run during an Xcode build to archive your build's
dSYM bundles (needed for symbolicating crash logs received from iTunes Connect
or beta testers' feedback) and, for beta releases, to create an `.ipa` package to
send to your beta testers.

By default, the dSYM bundle will be copied to and the `.ipa` will be created in
`${ARCHIVE_BASE}/${PROJECT_NAME}/${CONFIGURATION}/${ARCHIVE_DATE}/`.
* `ARCHIVE_BASE` is set at the top of the script and defaults to
  `${HOME}/dSYMArchive/`;
* `PROJECT_NAME` and `CONFIGURATION` are values provided by Xcode.
* `ARCHIVE_DATE` is the archive time in `YYYYMMDD-hhmmss` format.

## Installation

Because an `.ipa` must be created from already-signed binaries, you will need
to create a new Target in your Xcode project.

1. Right click on the _Targets_ group, select _Add > New Target_. Select an
   _Aggregate_ target from the _Other_ group. Name it and save it.
2. Bring up the Info inspector for your new aggregate target and add your
   executable(s) as direct dependencies for the target. Close the inspector.
3. Right click on your aggregate target and select _Add > New Build Phase > New
   Run Script Build Phase_. Set the _Shell_ to `/bin/sh` and the `Script` to a
   full path where you have placed this script. I keep it on a per-project
   basis in the `scripts` folder, so my path is
   `${PROJECT_DIR}/scripts/dsym-archiver`. You may optionally provide a
   base name for your `.ipa` in the event that your project name and the
   eventual release name do not match (as happened in the project where I built
   this script).

This new aggregate target should now be made your default build target to
obtain best results.

## What `dsym-archiver` Does
