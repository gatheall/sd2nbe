# DEPRECATED 

The session saving feature in Nessus was deprecated and removed a long time ago, before the release of Nessus 3.x. As a result, this script is likely to be of value only to those who have a collection of session data from ancient scans.


## Introduction

Nessus offers a feature known as session saving.  [NB: In versions of Nessus before 2.2.0, you had to enable this with the `--enable_save_sessions` option when running `nessus-core/configure`.] It's intended as a means to recover results of interrupted scans (due, for example, to a power outage or client machine crash), although it can also be used as a more general way of saving results.  Provided that the server was configured to support session saving and that the user elected to save the session when submitting a scan, session data will be saved in a directory such as `/usr/local/var/nessus/users/${user}/sessions`.

To recover results, a user connects to the Nessus server and restores the session.  The server then replays the session, scanning any hosts that were missed or unfinished from before, and displays the results.  One drawback to restoring a session, though, is the length of time the client takes to replay it.  Even if the scan was not interrupted, replaying the session may approach the time it took to do the scan originally.

**sd2nbe** takes an alternative approach -- it reads session data directly and outputs results in a format known as `NBE` (`Nessus BackEnd`).  While the format may not be especially readable, its use offers several attractive features:

* It can be fed into the unix-based nessus client and converted to a variety of other formats; eg, HTML, text, XML, etc (see nessus(1) for details).
* It can be easily filtered so as to limit reports, for example, to certain hosts or plugins.
* It can be merged with other NBE output simply by concatenating the sources.

**sd2nbe** is written in Perl.  It should work on any system with Perl 5 or better.  It also requires the Perl modules `Carp` and `Getopt::Long`.  If your system does not have these
modules installed already, visit [CPAN](http://search.cpan.org/) for help.


## Installation

* Retrieve [the script](sd2nbe) and save it locally.
* Verify ownership and permissions on the script - while only root will be able to read session data files, there's no reason why the script itself can't be accessed by another user.
* You may wish to edit the script to adjust the location of the perl interpreter in the first line.


## Use

This script operates like pretty much any other filter in unix -- reading input from either the file(s) named on the commandline or STDIN and writing output to STDOUT.  In the case of **sd2nbe**, input should be one or more session data files, which can be found in the directory `${prefix}/var/nessus/users/${user}/sessions`, provided session saving is in effect.

**sd2nbe** will also display copious debugging messages while running if the option `-d|--debug` is used.

Examples:

| Task | Commands |
| ---- | -------- |
| Convert a session from user 'theall' run on Jan 11 at 16:50 to NBE. | `sd2nbe /usr/local/var/nessus/users/theall/sessions/20040111-165004-data > session.nbe` |
| Convert the same session and display debugging messages. | `sd2nbe -d /usr/local/var/nessus/users/theall/sessions/20040111-165004-data > session.nbe` |

Examples of Manipulating Results in NBE Format:

| Task | Commands |
| ---- | -------- |
| Convert results.nbe to NSR format | `cat results.nbe | grep ^results | sed 's/^[^|]*|[^|]*|//g' > results.nsr` |
| Convert results.nbe to HTML with pie charts | `nessus -i results.nbe -o results.html_graph` |
| Filter results.nbe for one subnet and convert to XML | `fgrep '|10.0.1.|' results.nbe > subnet-10-0-1.nbe` |
| | `nessus -i subnet-10-0-1.nbe -o subnet-10-0-1.xml` |
| Produce an HTML report from results.nbe showing only critical / high risk vulnerabilities: [NB: only results with risk factors of _Critical_ and _High_ are output, not those such as _Low to High_.] | `egrep -i '(Risk|Risk +factor) *: *(Critical|High)' results.nbe > results-high.nbe` |
| | `nessus -i results-high.nbe -o results-high.html` |
| Create one HTML report for each host in results.nbe  | `awk -F'|' '$1 == "results" {hosts[$3]++} END {for (h in hosts) print h}' results.nbe > /tmp/hosts` |
| | `if test -s /tmp/hosts; then` |
| | `  for h in `cat /tmp/hosts`; do` |
| | `    fgrep "|$h|" results.nbe > $h.nbe` |
| | `    nessus -i $h.nbe -o $h.html` |
| | `    rm $h.nbe` |
| | `  done` |
| | `fi` |
| Extract scan results for just plugin #11921 from results.nbe and convert to HTML. | `awk -F'|' '$1 == "timestamps" || ($1 == "results" && $5 == "11921")' results.nbe > ms03-049.nbe` |
| | `nessus -i ms03-049.nbe -o ms03-049.html` |


## Known Bugs and Caveats

Currently, I am not aware of any bugs in this script.

Don't convert as a group multiple session data files that reflect scans of the same host(s); instead, convert them one at a time.

Warnings are issued for hosts that were not scanned completely. However, the script does not check whether the scan itself is complete (ie, all targets were tested).


## Copyright and License

Copyright (c) 2004-2016, George A. Theall.
All rights reserved.

This script is free software; you can redistribute it and/or modify it under the same terms as Perl itself.


