# Finnix mirrors

Finnix downloads are served by a network of community-sponsored mirrors; a full list is available at [mirrors.finnix.org](https://mirrors.finnix.org/).

## Official mirrors

Finnix is always welcoming new mirrors.
If you would like to become an official mirror, please contact us.

### When to mirror

Mirrors must be updated daily at the very least, but hourly would be preferred.
Usually the files don't change at all except before a release, so very little data will be transferred most of the time.

You can mirror any time of day, but please avoid mirroring directly on the hour.
Pick a random minute in cron to do the syncs.

### Server setup

* As of 2020, all new mirrors must serve HTTPS at least.
* Public rsync modules are nice to have, but not required.
* FTP and HTTP will be noted on [mirrors.finnix.org](https://mirrors.finnix.org/), but are otherwise not handled by the download redirector.

At the moment, only one primary rsync module is provided, finnix-releases.
Please do not pick a directory structure in the format `/pub/mirrors/finnix` to rsync this to, as future modules may be desired.
Please use a structure such as `/pub/mirrors/finnix/releases`. Almost all current mirrors have "releases" as the end directory.

Symlinks (relative) must be enabled on your web server software.
No dynamic content (CGI, etc) is served from the Finnix archive.

### Mirror script

The following script is recommended for performing the rsync itself.
You do not have to apply a trace file, but it is recommended.
However, you must never remove any of the files in the tree after transfer, especially the transferred trace files, as they are used by mirror monitoring scripts to determine if a mirror is out of date.

```shell
#!/bin/sh

# Needed variables
RSYNC_HOST="releases.finnix.org"
LOCAL_DEST="/path/to/pub/mirrors/finnix/releases"
LOCAL_HOST="$(hostname -f)"

# Perform the sync itself
rsync -a \
  rsync://${RSYNC_HOST}/finnix-releases/ \
  ${LOCAL_DEST}/

# Add a trace file.  Not needed, but highly recommended.
TZ=UTC date >"${LOCAL_DEST}/project/trace/${LOCAL_HOST}"
```

If you are mirroring Finnix for a private mirror, please do not use releases.finnix.org directly, and instead use a public mirror from [mirrors.finnix.org](https://mirrors.finnix.org/).

### Monitoring

Finnix will test your mirror every few hours to make sure it is functional.
The files in `/project/trace/` will be checked to determine mirror freshness, and small portions of release ISOs will be downloaded to verify server functionality.

Outdated mirrors will be removed from the automatic rotation until they are current again.
Mirrors that do not respond at all will be taken out of the rotation, and will be tested occasionally until they respond again.

## License

This document is provided under the following license:

    SPDX-PackageName: finnix-docs
    SPDX-PackageSupplier: Ryan Finnie <ryan@finnie.org>
    SPDX-PackageDownloadLocation: https://github.com/rfinnie/finnix-docs
    SPDX-FileCopyrightText: © 2021 Ryan Finnie <ryan@finnie.org>
    SPDX-License-Identifier: CC-BY-SA-4.0
