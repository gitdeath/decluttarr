_Like this app? Thanks for giving it a_ ⭐️

# **Decluttarr**

## Table of contents

-   [Overview](#overview)
-   [Dependencies & Hints & FAQ](#dependencies--hints--faq)
-   [Getting started](#getting-started)
-   [Explanation of the settings](#explanation-of-the-settings)
-   [Credits](#credits)
-   [Disclaimer](#disclaimer)

## Overview

Decluttarr keeps the radarr & sonarr & lidarr & readarr & whisparr queue free of stalled / redundant downloads

Feature overview:

-   Automatically delete downloads that are stuck downloading metadata (& trigger download from another source)
-   Automatically delete failed downloads (& trigger download from another source)
-   Automatically delete downloads belonging to radarr/sonarr/etc. items that have been deleted in the meantime ('Orphan downloads')
-   Automatically delete stalled downloads, after they have been found to be stalled multiple times in a row (& trigger download from another source)
-   Automatically delete slow downloads, after they have been found to be slow multiple times in a row (& trigger download from another source)
-   Automatically delete downloads belonging to radarr/sonarr/etc. items that are unmonitored
-   Automatically delete downloads that failed importing since they are not a format upgrade (i.e. a better version is already present)

You may run this locally by launching main.py, or by pulling the docker image.
You can find a sample docker-compose.yml [here](#method-1-docker).

## Dependencies & Hints & FAQ

-   Use Sonarr v4 & Radarr v5, else certain features may not work correctly
-   qBittorrent is recommended but not required. If you don't use qBittorrent, you will experience the following limitations:
    -   When detecting slow downloads, the speeds provided by the \*arr apps will be used, which is less accurate than what qBittorrent returns when queried directly
    -   The feature that allows to protect downloads from removal (NO_STALLED_REMOVAL_QBIT_TAG) does not work
    -   The feature that ignores private trackers does not work
-   If you see strange errors such as "found 10 / 3 times", consider turning on the setting "Reject Blocklisted Torrent Hashes While Grabbing". On nightly Radarr/Sonarr/Readarr/Lidarr/Whisparr, the option is located under settings/indexers in the advanced options of each indexer, on Prowlarr it is under settings/apps and then the advanced settings of the respective app
-   When broken torrents are removed the files belonging to them are deleted
-   Across all removal types: A new download from another source is automatically added by radarr/sonarr/lidarr/readarr/whisparr (if available)
-   If you use qBittorrent and none of your torrents get removed and the verbose logs tell that all torrents are protected by the NO_STALLED_REMOVAL_QBIT_TAG even if they are not, you may be using a qBittorrent version that has problems with API calls and you may want to consider switching to a different qBit image (see https://github.com/ManiMatter/decluttarr/issues/56)
-   Currently, “\*Arr” apps are only supported in English. Refer to issue https://github.com/ManiMatter/decluttarr/issues/132 for more details
-   If you experience yaml issues, please check the closed issues. There are different notations, and it may very well be that the issue you found has already been solved in one of the issues. Once you figured your problem, feel free to post your yaml to help others here: https://github.com/ManiMatter/decluttarr/issues/173
-   declutarr only supports single radarr / sonarr instances. If you have multiple instances of those \*arrs, solution is to run multiple decluclutarrs as well

## Getting started

There's two ways to run this:

-   As a docker container with docker-compose
-   By cloning the repository and running the script manually

Both ways are explained below and there's an explanation for the different settings below that

### Method 1: Docker

1. Make a `docker-compose.yml` file
2. Use the following as a base for that and tweak the settings to your needs

```yaml
version: "3.3"
services:
  decluttarr:
    image: ghcr.io/manimatter/decluttarr:latest
    container_name: decluttarr
    restart: always
    environment:
      TZ: Europe/Zurich
      PUID: 1000
      PGID: 1000

      ## General
      # TEST_RUN: True
      # SSL_VERIFICATION: False
      LOG_LEVEL: INFO

      ## Features
      REMOVE_TIMER: 10
      REMOVE_FAILED: True
      REMOVE_FAILED_IMPORTS: True
      REMOVE_METADATA_MISSING: True
      REMOVE_MISSING_FILES: True
      REMOVE_ORPHANS: True
      REMOVE_SLOW: True
      REMOVE_STALLED: True
      REMOVE_UNMONITORED: True
      RUN_PERIODIC_RESCANS: '
        {
        "SONARR": {"MISSING": true, "CUTOFF_UNMET": true, "MAX_CONCURRENT_SCANS": 3, "MIN_DAYS_BEFORE_RESCAN": 7},
        "RADARR": {"MISSING": true, "CUTOFF_UNMET": true, "MAX_CONCURRENT_SCANS": 3, "MIN_DAYS_BEFORE_RESCAN": 7}
        }'

      # Feature Settings
      PERMITTED_ATTEMPTS: 3
      NO_STALLED_REMOVAL_QBIT_TAG: Don't Kill
      MIN_DOWNLOAD_SPEED: 100
      FAILED_IMPORT_MESSAGE_PATTERNS: '
        [
        "Not a Custom Format upgrade for existing",
        "Not an upgrade for existing"
        ]'
      IGNORED_DOWNLOAD_CLIENTS: ["emulerr"]

      ## Radarr
      RADARR_URL: http://radarr:7878
      RADARR_KEY: $RADARR_API_KEY

      ## Sonarr
      SONARR_URL: http://sonarr:8989
      SONARR_KEY: $SONARR_API_KEY

      ## Lidarr
      LIDARR_URL: http://lidarr:8686
      LIDARR_KEY: $LIDARR_API_KEY

      ## Readarr
      READARR_URL: http://readarr:8787
      READARR_KEY: $READARR_API_KEY

      ## Whisparr
      WHISPARR_URL: http://whisparr:6969
      WHISPARR_KEY: $WHISPARR_API_KEY

      ## qBitorrent
      QBITTORRENT_URL: http://qbittorrent:8080
      # QBITTORRENT_USERNAME: Your name
      # QBITTORRENT_PASSWORD: Your password

```

3. Run `docker-compose up -d` in the directory where the file is located to create the docker container
   Note: Always pull the "**latest**" version. The "dev" version is for testing only, and should only be pulled when contributing code or supporting with bug fixes

### Method 2: Running manually

1. Clone the repository with `git clone -b latest https://github.com/ManiMatter/decluttarr.git`
Note: Do provide the `-b latest` in the clone command, else you will be pulling the dev branch which is not what you are after.
2. Rename the `config.conf-Example` inside the config folder to `config.conf`
3. Tweak `config.conf` to your needs
4. Install the libraries listed in the docker/requirements.txt (pip install -r requirements.txt)
5. Run the script with `python3 main.py`
   Note: The `config.conf` is disregarded when running via docker-compose.yml

## Explanation of the settings

### **General settings**

Configures the general behavior of the application (across all features)

**LOG_LEVEL**

-   Sets the level at which logging will take place
-   `INFO` will only show changes applied to radarr/sonarr/lidarr/readarr/whisparr
-   `VERBOSE` shows each check being performed even if no change is applied
-   `DEBUG` shows very granular information, only required for debugging
-   Type: String
-   Permissible Values: CRITICAL, ERROR, WARNING, INFO, VERBOSE, DEBUG
-   Is Mandatory: No (Defaults to INFO)

**TEST_RUN**

-   Allows you to safely try out this tool. If active, downloads will not be removed
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to False)

**SSL_VERIFICATION**

-   Turns SSL certificate verification on or off for all API calls
-   `True` means that the SSL certificate verification is on
-   Warning: It's important to note that disabling SSL verification can have security implications, as it makes the system vulnerable to man-in-the-middle attacks. It should only be done in a controlled and secure environment where the risks are well understood and mitigated
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to True)

---

### **Features settings**

Steers which type of cleaning is applied to the downloads queue

**REMOVE_TIMER**

-   Sets the frequency of how often the queue is checked for orphan and stalled downloads
-   Type: Integer
-   Unit: Minutes
-   Is Mandatory: No (Defaults to 10)

**REMOVE_FAILED**

-   Steers whether failed downloads with no connections are removed from the queue
-   These downloads are not added to the blocklist
    - A new download from another source is automatically added by radarr/sonarr/lidarr/readarr/whisparr (if available)
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to False)

**REMOVE_FAILED_IMPORTS**

-   Steers whether downloads that failed importing are removed from the queue
-   This can happen, for example, when a better version is already present
-   Note: Only considers an import failed if the import message contains a warning that is listed on FAILED_IMPORT_MESSAGE_PATTERNS (see below)
-   These downloads are added to the blocklist
-   If the setting IGNORE_PRIVATE_TRACKERS is true, and the affected torrent is a private tracker, the queue item will be removed, but the torrent files will be kept
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to False)

**REMOVE_METADATA_MISSING**

-   Steers whether downloads stuck obtaining metadata are removed from the queue
-   These downloads are added to the blocklist, so that they are not re-requested
-   A new download from another source is automatically added by radarr/sonarr/lidarr/readarr/whisparr (if available)
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to False)

**REMOVE_MISSING_FILES**

-   Steers whether downloads that have the warning "Files Missing" are removed from the queue
-   These downloads are not added to the blocklist
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to False)

**REMOVE_ORPHANS**

-   Steers whether orphan downloads are removed from the queue
-   Orphan downloads are those that do not belong to any requested media anymore (Since the media was removed from radarr/sonarr/lidarr/readarr/whisparr after the download started)
-   These downloads are not added to the blocklist
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to False)

**REMOVE_SLOW**

-   Steers whether slow downloads are removed from the queue
-   Slow downloads are added to the blocklist, so that they are not re-requested in the future
-   Note: Does not apply to usenet downloads (since there users pay for certain speed, slowness should not occurr)
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to False)

**REMOVE_STALLED**

-   Steers whether stalled downloads with no connections are removed from the queue
-   These downloads are added to the blocklist, so that they are not re-requested in the future
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to False)

**REMOVE_UNMONITORED**

-   Steers whether downloads belonging to unmonitored media are removed from the queue
-   Note: Will only remove from queue if all TV shows depending on the same download are unmonitored
-   These downloads are not added to the blocklist
-   Note: Since sonarr does not support multi-season packs, if you download one you should protect it with `NO_STALLED_REMOVAL_QBIT_TAG` that is explained further down
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to False)

**RUN_PERIODIC_RESCANS**

-   Steers whether searches are automatically triggered for items that are missing or have not yet met the cutoff
-   Note: Only supports Radarr/Sonarr currently (Lidarr depending on: https://github.com/Lidarr/Lidarr/pull/5084 / Readarr Depending on: https://github.com/Readarr/Readarr/pull/3724)
-   Type: Dictionaire
-   Is Mandatory: No (Defaults to no searches being triggered automatically)
-   "SONARR"/"RADARR" turns on the automatic searches for the respective instances
-   "MISSING"/"CUTOFF_UNMET" turns on the automatic search for those wanted items (defaults to True)
-   "MAX_CONCURRENT_SCANS" specifies the maximum number of items to be searched in each scan. This value dictates how many items are processed per search operation, which occurs according to the interval set by the REMOVE_TIMER.
-   Note: The limit is per wanted list. Thus if both Radarr & Sonarr are set up for automatic searches, both for missing and cutoff unmet items, the actual count may be four times the MAX_CONCURRENT_SCANS
-   "MIN_DAYS_BEFORE_RESCAN" steers the days that need to pass before an item is considered again for a scan
-   Note: RUN_PERIODIC_RESCANS will always search those items that haven been searched for longest

```
     RUN_PERIODIC_RESCANS: '
        {
          "SONARR": {"MISSING": true, "CUTOFF_UNMET": true, "MAX_CONCURRENT_SCANS": 3, "MIN_DAYS_BEFORE_RESCAN": 7},
          "RADARR": {"MISSING": true, "CUTOFF_UNMET": true, "MAX_CONCURRENT_SCANS": 3, "MIN_DAYS_BEFORE_RESCAN": 7}
        }'
```

There are different yaml notations, any some users suggested the below alternative notation.
If it you face issues, please first check the closed issues before opening a new one (e.g., https://github.com/ManiMatter/decluttarr/issues/173)

```
- RUN_PERIODIC_RESCANS=[
{
"SONARR":[{"MISSING":true, "CUTOFF_UNMET":true, "MAX_CONCURRENT_SCANS":3, "MIN_DAYS_BEFORE_RESCAN":7}],
"RADARR":[{"MISSING":true, "CUTOFF_UNMET":true, "MAX_CONCURRENT_SCANS":3, "MIN_DAYS_BEFORE_RESCAN":7}]
}
```

**MIN_DOWNLOAD_SPEED**

-   Sets the minimum download speed for active downloads
-   If the increase in the downloaded file size of a download is less than this value between two consecutive checks, the download is considered slow and is removed if happening more ofthen than the permitted attempts
-   Type: Integer
-   Unit: KBytes per second
-   Is Mandatory: No (Defaults to 100, but is only enforced when "REMOVE_SLOW" is true)

**PERMITTED_ATTEMPTS**

-   Defines how many times a download has to be caught as stalled, slow or stuck downloading metadata before it is removed
-   Type: Integer
-   Unit: Number of scans
-   Is Mandatory: No (Defaults to 3)

**NO_STALLED_REMOVAL_QBIT_TAG**

-   Downloads in qBittorrent tagged with this tag will not be removed
-   Feature is not available when not using qBittorrent as torrent manager
-   Applies to all types of removal (ie. nothing will be removed automatically by decluttarr)
-   Note: You may want to try "force recheck" to get your stuck torrents manually back up and running
-   Tag is automatically created in qBittorrent (required qBittorrent is reachable on `QBITTORRENT_URL`)
-   Important: Also protects unmonitored downloads from being removed (relevant for multi-season packs)
-   Type: String
-   Is Mandatory: No (Defaults to `Don't Kill`)

**IGNORE_PRIVATE_TRACKERS**

-   Private torrents in qBittorrent will not be removed from the queue if this is set to true
-   Only works if qBittorrent is used (does not work with transmission etc.)
-   Applies to all types of removal (ie. nothing will be removed automatically by decluttarr); only exception to this is REMOVE_NO_FORMAT_UPGRADE, where for private trackers the queue item is removed (but the torrent files are kept)
-   Note: You may want to try "force recheck" to get your stuck torrents manually back up and running
-   Type: Boolean
-   Permissible Values: True, False
-   Is Mandatory: No (Defaults to True)

**FAILED_IMPORT_MESSAGE_PATTERNS**

-   Works in together with REMOVE_FAILED_IMPORTS (only relevant if this setting is true)
-   Defines the patterns based on which the tool decides if a completed download that has warnings on import should be considered failed
-   Queue items are considered failed if any of the specified patterns is contained in one of the messages of the queue item
-   Note: If left empty (or not specified), any such pending import with warning is considered failed
-   Type: List
-   Recommended values: ["Not a Custom Format upgrade for existing", "Not an upgrade for existing"]
-   Is Mandatory: No (Defaults to [], which means all messages are failures)

**IGNORED_DOWNLOAD_CLIENTS**

- If specified, downloads of the listed download clients are not removed / skipped entirely
- Is useful if multiple download clients are used and some of them are known to have slow downloads that recover (and thus should not be subject to slowness check), while other download clients should be monitored
- Type: List
- Is Mandatory: No (Defaults to [], which means no download clients are skipped)

---

### **Radarr section**

Defines radarr instance on which download queue should be decluttered

**RADARR_URL**

-   URL under which the instance can be reached
-   If not defined, this instance will not be monitored

**RADARR_KEY**

-   Your API key for radarr

---

### **Sonarr section**

Defines sonarr instance on which download queue should be decluttered

**SONARR_URL**

-   URL under which the instance can be reached
-   If not defined, this instance will not be monitored

**SONARR_KEY**

-   Your API key for sonarr

---

### **Lidarr section**

Defines lidarr instance on which download queue should be decluttered

**LIDARR_URL**

-   URL under which the instance can be reached
-   If not defined, this instance will not be monitored

**LIDARR_KEY**

-   Your API key for lidarr

---

### **Readarr section**

Defines readarr instance on which download queue should be decluttered

**READARR_URL**

-   URL under which the instance can be reached
-   If not defined, this instance will not be monitored

**READARR_KEY**

-   Your API key for readarr

---

### **Whisparr section**

Defines whisparr instance on which download queue should be decluttered

**WHISPARR_URL**

-   URL under which the instance can be reached
-   If not defined, this instance will not be monitored

**WHISPARR_KEY**

-   Your API key for whisparr

---

### **qBittorrent section**

Defines settings to connect with qBittorrent
If a different torrent manager is used, comment out this section (see above the limitations in functionality that arises from this)

**QBITTORRENT_URL**

-   URL under which the instance can be reached
-   If not defined, the NO_STALLED_REMOVAL_QBIT_TAG takes no effect

**QBITTORRENT_USERNAME**

-   Username used to log in to qBittorrent
-   Optional; not needed if authentication bypassing on qBittorrent is enabled (for instance for local connections)

**QBITTORRENT_PASSWORD**

-   Password used to log in to qBittorrent
-   Optional; not needed if authentication bypassing on qBittorrent is enabled (for instance for local connections)

## Credits

-   Script for detecting stalled downloads expanded on code by MattDGTL/sonarr-radarr-queue-cleaner
-   Script to read out config expanded on code by syncarr/syncarr
-   SONARR/RADARR team & contributors for their great product, API documenation, and guidance in their Discord channel
-   Particular thanks to them for adding an additional flag to their API that allowed this script detect downloads stuck finding metadata
-   craggles17 for arm compatibility
-   Fxsch for improved documentation / ReadMe

## Disclaimer

This script comes free of any warranty, and you are using it at your own risk
