# Finnix release procedure

## Variables

Gather the variables used throughout this document:

```shell
FINNIX_VER=999
FINNIX_ARCH=amd64
FINNIX_RELEASE_DATE=2099-01-01
FINNIX_NEXT_CODENAME=hi
```

## Release fitness

Check the [Finnix issue tracker](https://github.com/finnix/finnix/issues) and see if there are any issues which can be solved before the release.

Debian packages are removed from the Finnix package lists because they drop out of Debian testing, usually due to RC issues. Check for any removed packages which re-appeared in Debian but were not added back to Finnix.

## SquashFS sort files

Boot a release-quality build (i.e. as close to what will be released as possible, but doesn't have to be 100% exact), with `init.strace=1` appended to the kernel command line.

Wait about 10 seconds after booting is complete, then run:

```shell
killall strace
git clone --depth=1 https://github.com/finnix/finnix-live-build
finnix-live-build/tools/strace-reorder </var/log/strace-init.trace >squashfs.sort
```

`strace-reorder` must be done on the live system as it must be able to examine the root filesystem it was run on, to dereference symlinks, etc.

Copy `squashfs.sort` off the test machine.

## Release build

Branch finnix-live-build main to e.g. `v${FINNIX_VER?}-rc`.  The branch name is not critical but will be temporarily pushed publicly, so this is a good format. `v${FINNIX_VER?}` alone should not be used because that is the tag name.

```shell
git pull
git checkout -b v${FINNIX_VER?}-rc
```

Edit `finnix-live-build`, change:

* `VERSION` to the final number
* `SAVE_ISO` to true
* `CACHE_FROZEN` to true

Place `squashfs.sort` as `files/squashfs.${FINNIX_VER?}.${FINNIX_ARCH?}.sort`.

```shell
cp /tmp/squashfs.sort files/squashfs.${FINNIX_VER?}.${FINNIX_ARCH?}.sort
cat >"files/squashfs.${FINNIX_VER?}.${FINNIX_ARCH?}.sort.license" <<"EOM"
SPDX-PackageSummary: finnix-live-build
SPDX-FileCopyrightText: None
SPDX-License-Identifier: CC0-1.0
EOM

git add .
git commit -m "Finnix ${FINNIX_VER?}"
git push origin v${FINNIX_VER?}-rc
```

Run the [release workflow](https://github.com/finnix/finnix-live-build/actions/workflows/release.yml), making sure to run the workflow against the RC branch, and download the built artifacts.

### Docker riscv64 build

*Still not sure if riscv64 will be part of the final release manifest, as it must be built manually.*

On `flexo.snowman.lan` (StarFive VisionFive 2):

```shell
git fetch origin v${FINNIX_VER?}-rc
git checkout origin/v${FINNIX_VER?}-rc
env DOCKER_BUILD="true" ./finnix-live-build
docker image pull docker.io/library/debian:testing
docker image build -t docker.io/finnix/finnix:rc-riscv64 build/docker
```

## Release testing

* Default UEFI VM boot
* Default BIOS VM boot
* Real hardware - USB
* Real hardware - CD
* UEFI secure boot
* Kernel command line: `sshd passwd=foo`
* Kernel command line: `toram`
* `wifi-connect` usability

## Signatures

Retrieve OpenPGP and SSH keys from the vault.

### OpenPGP

```shell
gpg --import 867279B83BF8E815A236D39E7D6F85C04356E6C2.private.asc

gpg --armor --detach-sign --local-user "Finnix Release Signing Key <keymaster@finnix.org>" finnix-${FINNIX_VER?}.iso
mv finnix-${FINNIX_VER?}.iso.asc finnix-${FINNIX_VER?}.iso.gpg
gpg --armor --detach-sign --local-user "Finnix Release Signing Key <keymaster@finnix.org>" finnix-${FINNIX_VER?}-source.iso
mv finnix-${FINNIX_VER?}-source.iso.asc finnix-${FINNIX_VER?}-source.iso.gpg
```

ASCII-armored OpenPGP detached signatures keep the filename `.gpg` for historical reasons.

Remove `~/.gnupg` from the temporary user account.

### SSH

```shell
ssh-keygen -Y sign -f finnix-file/id_ed25519 -n file finnix-${FINNIX_VER?}.iso
```

This creates `finnix-${FINNIX_VER?}.iso.sig`; keep for release data generation.

Remove the directory `finnix-file/`.

## Torrent

### File creation

```shell
mktorrent \
    --announce=https://tracker.finnix.org/announce \
    --announce=https://ipv6.tracker.finnix.org/announce \
    --comment=finnix-${FINNIX_VER?}.iso \
    --output=finnix-${FINNIX_VER?}.iso.torrent \
    --web-seed=https://mirror.aarnet.edu.au/pub/finnix/${FINNIX_VER?}/finnix-${FINNIX_VER?}.iso \
    --web-seed=https://ftp.cc.uoc.gr/mirrors/linux/finnix/releases/${FINNIX_VER?}/finnix-${FINNIX_VER?}.iso \
    --web-seed=https://mirrors.catalyst.net.nz/finnix-releases/${FINNIX_VER?}/finnix-${FINNIX_VER?}.iso \
    --web-seed=https://mirrors.ocf.berkeley.edu/finnix-releases/${FINNIX_VER?}/finnix-${FINNIX_VER?}.iso \
    finnix-${FINNIX_VER?}.iso
```

Web seed mirrors help, but are not critical to remain fresh/reliable.  Torrent files can be regenerated and replaced in the archive without problem to change web seeds; they will retain the same hash ID.

### Tracker

Get the hex hash of the torrent:

```shell
btcheck -i -l finnix-${FINNIX_VER?}.iso.torrent
```

Edit [finnix-tracker](https://github.com/finnix/finnix-tracker)/`finnix-tracker.js`, add to `allowedHashes`.

Commit, push, [build Docker image](https://github.com/finnix/finnix-tracker/actions/workflows/registry.yml), restart container.

## Upload

### Archive

The release directory should look like so:

```shell
${FINNIX_VER?}/finnix-${FINNIX_VER?}.iso
${FINNIX_VER?}/finnix-${FINNIX_VER?}.iso.gpg
${FINNIX_VER?}/finnix-${FINNIX_VER?}.iso.torrent
```

Upload it to the main release box in a separate directory (home directory), so mirrors don't see the release mid-upload. Then:

```shell
mv ~/${FINNIX_VER?} /srv/www/releases.finnix.org/htdocs/finnix-releases/
rm -f /srv/www/releases.finnix.org/htdocs/finnix-releases/current
ln -s ${FINNIX_VER?} /srv/www/releases.finnix.org/htdocs/finnix-releases/current
```

This should be done 24-36 hours before release announcement, to allow time for the community mirrors to complete.

### Internet Archive

```shell
ia upload finnix_${FINNIX_VER?}_source \
    finnix-${FINNIX_VER?}-source.iso finnix-${FINNIX_VER?}-source.iso.gpg finnix-simple-sleeve.jpg \
    --metadata="collection:finnix" \
    --metadata="creator:Ryan Finnie" \
    --metadata="date:${FINNIX_RELEASE_DATE?}" \
    --metadata="description:This CD contains the sources used to build Finnix ${FINNIX_VER?}. It is not officially released by Finnix, but is publicly available for license compliance and historical preservation." \
    --metadata="language:English" \
    --metadata="licenseurl:https://www.mozilla.org/en-US/MPL/2.0/" \
    --metadata="mediatype:software" \
    --metadata="possible-copyright-status:MPL 2.0 classification refers to the release as a whole; release contains numerous 3rd-party software released under a variety of open source licenses." \
    --metadata="source:https://www.finnix.org/" \
    --metadata="subject:finnix" \
    --metadata="subject:unreleased" \
    --metadata="subject:source" \
    --metadata="title:Finnix ${FINNIX_VER?} source CD"

ia upload finnix_${FINNIX_VER?} \
    finnix-${FINNIX_VER?}.iso finnix-${FINNIX_VER?}.iso.gpg finnix-simple-sleeve.jpg \
    --metadata="collection:finnix" \
    --metadata="creator:Ryan Finnie" \
    --metadata="date:${FINNIX_RELEASE_DATE?}" \
    --metadata="description:Finnix ${FINNIX_VER?}, all officially released disc images and accompanying files." \
    --metadata="language:English" \
    --metadata="licenseurl:https://www.mozilla.org/en-US/MPL/2.0/" \
    --metadata="mediatype:software" \
    --metadata="possible-copyright-status:MPL 2.0 classification refers to the release as a whole; release contains numerous 3rd-party software released under a variety of open source licenses." \
    --metadata="source:https://www.finnix.org/" \
    --metadata="subject:finnix" \
    --metadata="title:Finnix ${FINNIX_VER?}"
```

## Docker

```shell
docker image pull ghcr.io/finnix/finnix-live-build:release-amd64
docker image pull ghcr.io/finnix/finnix-live-build:release-arm64
ssh flexo.snowman.lan docker image save docker.io/finnix/finnix:rc-riscv64 | docker image load
docker image tag ghcr.io/finnix/finnix-live-build:release-amd64 docker.io/finnix/finnix:rc-amd64
docker image tag ghcr.io/finnix/finnix-live-build:release-arm64 docker.io/finnix/finnix:rc-arm64
docker image tag docker.io/finnix/finnix:riscv64-latest docker.io/finnix/finnix:rc-riscv64
docker image push docker.io/finnix/finnix:rc-amd64
docker image push docker.io/finnix/finnix:rc-arm64
docker image push docker.io/finnix/finnix:rc-riscv64

docker manifest rm docker.io/finnix/finnix:${FINNIX_VER?} || true
docker manifest rm docker.io/finnix/finnix:latest || true
docker manifest create docker.io/finnix/finnix:${FINNIX_VER?} \
    docker.io/finnix/finnix:rc-amd64 \
    docker.io/finnix/finnix:rc-arm64 \
    docker.io/finnix/finnix:rc-riscv64
docker manifest create docker.io/finnix/finnix:latest \
    docker.io/finnix/finnix:rc-amd64 \
    docker.io/finnix/finnix:rc-arm64 \
    docker.io/finnix/finnix:rc-riscv64
docker manifest push docker.io/finnix/finnix:${FINNIX_VER?}
docker manifest push docker.io/finnix/finnix:latest
```

Afterward, go into the Docker Hub interface and untag the `rc` tags.

## Release data

Make sure the OpenPGP signature (`finnix-${FINNIX_VER?}.iso.gpg`) and SSH signature (`finnix-${FINNIX_VER?}.iso.sig`) are in the same directory as the ISO. Then, in the [finnix-docs](https://github.com/finnix/finnix-docs) clone:

```shell
tools/make-release-json --release-date=${FINNIX_RELEASE_DATE?} finnix-${FINNIX_VER?}.iso >releases/${FINNIX_VER?}.json
```

Double check the JSON file, add and commit it.

Push to origin a few hours after the release is on the primary archive, but before release.

## Finalize branch

Tag and push the RC commit:

```shell
git checkout main
git merge v${FINNIX_VER?}-rc
git tag -m "Finnix ${FINNIX_VER?}" v${FINNIX_VER?}
git push origin --tags
```

Update the following changes in `finnix-live-build` to go back to dev:

* `VERSION` to dev
* `SAVE_ISO` to false
* `CACHE_FROZEN` to false
* `CODENAME` to the next codename

```shell
rm -f files/squashfs.${FINNIX_VER?}.${FINNIX_ARCH?}.sort
git add .
git commit -m "Finnix dev (${FINNIX_NEXT_CODENAME?})"
git push
git branch -d v${FINNIX_VER?}-rc
```

Note that earlier, `git push origin --tags` does not actually push to main, just pushes the tag. The branch push doesn't occur until after the dev commit, after which point main advances by 2 commits (release then dev). This is desired to not have any automated opportunistic builds happen against the release commit itself (besides the release workflow we trigger against the RC branch, that is).

## Documentation / site updates

* Homepage
* Blog release announcement
* Social posts

## Issue management

Go to the [Finnix issue tracker](https://github.com/finnix/finnix/issues) and close out:

* Any committed (fixed) issues
* The milestone tracking issue
* The milestone itself

Open a new milestone and milestone tracking issue for the next release.

## Mirrors

The [Finnix mirror network](https://mirrors.finnix.org/) tests using release ISOs, generally the latest release plus Finnix 109.  The Django settings should be updated for the new release a day or so after all the mirrors have synced (not before, as the checker would error out on a mirror about missing files).

For the example configuration in [finnix-mirrors-website](https://github.com/finnix/finnix-mirrors-website):

```shell
utils/iso_randchunk_hashes \
    --position=1048576 \
    --path-base=releases \
    releases/${FINNIX_VER?}/finnix-${FINNIX_VER?}.iso \
    | black -q -
```

But for the actual site, we have it pick 128 random chunks to hash, which `iso_randchunk_hashes` does by default:

```shell
utils/iso_randchunk_hashes \
    --path-base=releases \
    releases/${FINNIX_VER?}/finnix-${FINNIX_VER?}.iso \
    | black -q -
```

## Cleanup

Double check GPG/SSH keys have been deleted.

Delete the RC branch on the GitHub remote repository.

Run a concurrent build on the personal build environment, then update the "focus" symlink:

```shell
sudo ./finnix-live-build
sudo rm -f build/info/focus
sudo ln -s $(readlink build/info/previous) build/info/focus
```

## Release checklist

Paste into milestone tracking ticket.

* [ ] [Release fitness](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#release-fitness)
* [ ] [SquashFS sort files](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#squashfs-sort-files)
* [ ] [Release build](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#release-build)
* [ ] [Release source build](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#release-source-build)
* [ ] [Release testing](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#release-testing)
  * [ ] Default UEFI VM boot
  * [ ] Default BIOS VM boot
  * [ ] Real hardware - USB
  * [ ] Real hardware - CD
  * [ ] UEFI secure boot
  * [ ] Kernel command line: `sshd passwd=foo`
  * [ ] Kernel command line: `toram`
  * [ ] `wifi-connect` usability
* [ ] [Signatures](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#signatures)
  * [ ] OpenPGP
  * [ ] SSH
* [ ] [Torrent](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#torrent)
  * [ ] File creation
  * [ ] Tracker
* [ ] [Upload](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#upload)
  * [ ] Archive
  * [ ] Internet Archive
* [ ] [Release data](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#release-data)
  * [ ] Build
  * [ ] Push
* [ ] [Finalize branch](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#finalize-branch)
* [ ] [Documentation / site updates](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#documentation--site-updates)
* [ ] [Issue management](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#issue-management)
* [ ] [Mirrors](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#mirrors)
* [ ] [Cleanup](https://github.com/finnix/finnix-docs/blob/main/release-procedure.md#cleanup)

## License

This document is provided under the following license:

    SPDX-PackageSummary: finnix-docs
    SPDX-FileCopyrightText: © 2021 Ryan Finnie <ryan@finnie.org>
    SPDX-License-Identifier: CC-BY-SA-4.0
