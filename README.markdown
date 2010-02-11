# `dsym-archiver` README

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

You may change `ARCHIVE_BASE` at the top of the script if `${HOME}/dsymArchive`
is not to your liking. This location _must_ be in a location accessible to
spotlight.

See a presentation I gave at TACOW in January 2010 for more information (slides
17-24). [Solving Little Problems][http://www.slideshare.net/halostatue/solving-little-things/17]

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

This script is fairly simple, but fairly verbose because it's using environment
variables from Xcode to provide most of the magic involved.

1. If this build script is run and the `SDK_NAME` does not start with
   `iphoneos`, the script does nothing (and prints an error if you run the
   script outside of Xcode).

      case @${SDK_NAME} in
        @iphoneos*)   archive_dsym_iphoneos "${1}" ;;
        @*)           : ;;
        @)            echo "Not running under Xcode." ;;
      esac

2. If you're building an iPhone OS project, and you're running a build
   `CONFIGURATION` containing the term `Distribution`, it copies the dSYM
   debugging symbols from their target folder (`DWARF_DSYM_FOLDER_PATH`) to the
   `ARCHIVE_FOLDDER` (described above).

      case ${CONFIGURATION} in
        *Distribution*) : ;;
        *)              return ;;
      esac

3. If step two has run, the script checks to see if you are using a `Beta` (or
   `beta`) build `CONFIGURATION`. If you are, the `.app` folders are copied to
   a temporary location; if there is an `iTunesArtwork` file in the `.app`
   folder, this is copied out as well. These are then zipped into an `.ipa`
   archive and the `ARCHIVE_FOLDER` is opened in iTunes.

      case ${CONFIGURATION} in
        *[Bb]eta*)  prepare_adhoc_package "${1}" ;;
        *)          return ;;
      esac

