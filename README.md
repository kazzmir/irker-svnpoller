irker-svnpoller
===============

A generic SVN poller script that reads from a SVN repository and delivers
notifications to an [irkerd(1)][irkerd] server running on localhost.

This script requires no configuration. See **irker-svnpoller --help** for
command line usage information.

[irkerd]: http://www.catb.org/esr/irker/


svnmail-filter
==============

This is a mail filter script that reads an email message from stdin, matches
it against rulesets intended for SVN commit mailing lists, and invokes the SVN
poller for matching messages.

It supports multiple projects, each one associated to a unique mailing list and
SVN repository. Notifications may be delivered to a single channel, or to
multiple channels, and they may even be delivered to additional channels for
specific authors.

It’s primarily intended for use with **procmail** or any other method that
allows filtering incoming mail through a script.

Example .procmailrc and filter ruleset files are provided in the distribution.
Some additional configuration may be required by editing the script if its
dependencies are not located on the same location as **svnmail-filter** itself.


svnmail-rules.json structure
============================

*To be documented.*
