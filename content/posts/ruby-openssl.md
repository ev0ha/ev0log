---
title: "Fixing Ruby OpenSSL issue on Ubuntu"
date: 2022-10-14T17:22:16+02:00
draft: true
---

# It seems there is a conflict between OpenSSL versions used by Ruby/gems and the one installed by default on Ubuntu

While trying to install a gem I got this error:

```bash
ERROR : While executing gem … (Gem::Exception) Unable to require openssl, install OpenSSL and rebuild ruby (preferred)
or use non-HTTPS sources $gem source -l
```

But OpenSSL is certainly already installed. After trying to troubleshoot the issue further by searching for workarounds
on Stackoverflow and Github, I couldn't make it work. Rebuilding Ruby was my last resort, so I tried one last thing:
Uninstall everything Ruby related completely from the system (which obviously breaks the system Ruby installation,
so proceed with caution and on your own risk) and install the packages again. After uninstalling, take a look
at history.log under var/log/apt:
```
Start-Date: 2022-10-30  11:00:31
Commandline: apt remove ruby
Requested-By: ev0 (1000)
Remove: ruby-zeitwerk:amd64 (2.4.2-2), ruby-thor:amd64 (1.2.1-1), ruby-railties:amd64 (2:6.1.7+dfsg-1), ruby-crass:amd64 (1.0.2-3), libruby3.0:amd64 (3.0.4-7), racc:amd64 (1.4.14-2.1), rake:amd64 (13.0.6-3), ruby:amd64 (1:3.0~exp1), ruby-erubi:amd64 (1.9.0-2), ruby-rails-deprecated-sanitizer:amd64 (1.0.3-3.1), ruby-rails-html-sanitizer:amd64 (1.4.2-2), ruby-loofah:amd64 (2.18.0-1), ruby3.0:amd64 (3.0.4-7), ruby-actionpack:amd64 (2:6.1.7+dfsg-1), ruby-rack-test:amd64 (2.0.2-1), ruby-rack:amd64 (2.2.4-2), ruby-rubygems:amd64 (3.3.15-1), ruby-actionview:amd64 (2:6.1.7+dfsg-1), ruby-rails-dom-testing:amd64 (2.0.3-4), ruby-activesupport:amd64 (2:6.1.7+dfsg-1), ruby-pkg-config:amd64 (1.4.6-4), ruby-nokogiri:amd64 (1.13.7+dfsg-2)
End-Date: 2022-10-30  11:00:32
```
Now we install all the removed packages manually ... and it works, gem installs successfully, yay!
