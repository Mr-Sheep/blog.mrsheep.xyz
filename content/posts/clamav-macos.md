---
title: "ClamAV MacOS X"
date: 2020-10-13T14:05:05+08:00
draft: false
tags: ["V2ray", "HTML"]
categories: ["anti-censorship"]
---

One of the best open source anti-virus for Mac OS X is [ClamAV](www.clamav.net) by Cisco.
ClamAV is a crossplatform open source anti-virus with high performance, it is also the open source standard for mail gateway scanning software.

<!--more-->
The most easy and efficient way to install clamAV on OS X is using [homebrew](http://brew.sh/)
`brew install clamav`
Before you could scan any file, first do the following:
Under `/usr/local/etc/clamav`
```
cp freshclam.conf.sample freshclam.conf
cp clamd.conf.sample clamd.conf

```
and comment out following lines in both files:
```
# Comment or remove the line below.
# Example
```
and add the mirror line to clamd.conf:
```
DatabaseMirror database.clamav.net
```
Then run: `freshclam` to update the database

Now, you could use `clamscan -r --bell -i /path/to/file` to scan the file, where:

```
    --help                -h             Show this help
    --version             -V             Print version number
    --verbose             -v             Be verbose
    --archive-verbose     -a             Show filenames inside scanned archives
    --debug                              Enable libclamav's debug messages
    --quiet                              Only output error messages
    --stdout                             Write to stdout instead of stderr. Does not affect 'debug' messages.
    --no-summary                         Disable summary at end of scanning
    --infected            -i             Only print infected files
    --suppress-ok-results -o             Skip printing OK files
    --bell
```