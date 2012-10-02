irker-svnpoller
===============

A generic SVN poller script that reads from a SVN repository and delivers
notifications to an **[irkerd(1)][irkerd]** server running on localhost.

This script requires no configuration. See **irker-svnpoller --help** for
command line usage information.

[irkerd]: http://www.catb.org/esr/irker/


svnmail-filter
==============

This is a mail filter script that reads an email message from stdin, matches
it against rulesets intended for SVN commit mailing lists, and invokes
**irker-svnpoller** for matching messages. This is primarily intended as a
stopgap solution for projects using SVN repositories where installing or
reconfiguring hooks is not possible, but still have a commits mailing list to
which new accounts can be subscribed.

It supports multiple projects, each one associated to a unique mailing list and
SVN repository. Notifications may be delivered to a single channel, or to
multiple channels, and they may even be delivered to additional channels for
specific authors.

It’s primarily intended for use with **procmail(1)** or any other method that
allows filtering incoming mail through a script.

Example .procmailrc and filter ruleset files are provided in the distribution.
Some additional configuration may be required by editing the script if its
dependencies are not located on the same location as **svnmail-filter** itself.


Ruleset file
------------

svnmail-filter rulesets are read from **svnmail-rules.json** on the same
location as the script itself with the default configuration.

Rulesets are objects associated to individual mailing list addresses:

```JSON
{
    "commit-mailing-list-1@example.org": { <mailing list rules object> },
    "commit-mailing-list-2@example.org": { <mailing list rules object> },
    <...>
}
```

svnmail-filter will only process email coming from the configured addresses.
Each address is associated to an object describing how to process incoming mail
from it, including the SVN repository to poll from, and the channels to which
notifications will be delivered.

These objects are structured as follows:

```JSON
{
    "pattern": <regular expression>,
    "project": <project id string>,
    "repository": <repository URI string>,
    "channels" : { <channel delivery rules object> }
}
```

 * **pattern**: A regular expression pattern used to capture the SVN revision
   number from email subjects. The first group (parentheses) is captured and
   assumed to be a valid revision number.

   Since the subject format varies across different repositories and mailing
   lists, you will most certainly need to specify different patterns for
   different addresses.

   Note that backslashes in escape sequences (as in `\[project name\]` to match
   literal brackets) need to be escaped for JSON too, e.g.
   `\\project name\\]`.

 * **project**: The project id/name used for `irkerd` notifications via
   `irker-svnpoller`.

 * **repository**: The URI for the SVN repository. Commit logs are pulled from
   this URI. **It must be an authentication-free repository**, usually using
   the `http://` or `svn://` transports.

 * **channels**: An object containing channel delivery rules.
   ```JSON
   {
       "*": [
           "irc://irc.freenode.net/#awesome-project",
           "irc://irc.freenode.net/#commits"
       ],
       "individual-author@example.com": [
           "irc://irc.freenode.net/##authors-channel",
       ],
       "author-email-username": [
           "irc://irc.freenode.net/##randomness",
       ]
   }
   ```
   Each entry is associated to an author match pattern, which must be either
   `*` (for all authors), a complete author email address, or just the username
   portion of an email address. Commits are delivered to the lists of channels
   specified for each match.

   It is possible for a commit to match multiple rules. For example, a commit
   reported to the mailing list for john@example.com will match the `*`,
   `john@example.com`, and `john` rules at the same time, and notifications
   will be delivered to all channels specified for each one.
