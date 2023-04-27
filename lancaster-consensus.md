# The Lancaster Consensus

At the first Perl QA Hackathon in 2008 in Oslo, a number of QA and
toolchain authors, maintainers and experts came together to agree on some
common standards and practices.  This became known as
["The Oslo Consensus"](https://github.com/Perl-Toolchain-Gang/toolchain-site/blob/master/oslo-consensus.md).

Five years later, at the 2013 Perl QA Hackathon, a similar brain trust came
together to address new issues requiring consensus.

These decisions provide direction, but, as always, the speed of
implementation will depend on the interests and availability of volunteers
to do the work.

## Toolchain and testing

### Minimum-supported Perl

> 👉 This section is obsoleted by the [Lyon Amendment](lyon-amendment.md) to
> the Lancaster Consensus.  See that document for more details.

Going forward, the Perl toolchain will target Perl 5.8.1, released
September 2003.  This will allow toolchain modules to reliably use PerlIO
and improved Unicode support.

Because of the many Unicode bug-fixes in early 5.8 releases,
toolchain maintainers reserve the right to later bump the
minimum to 5.8.4 (which ships with Solaris 10).

### Specifying pure-perl builds

Some distributions offer an "XS" version or a "Pure Perl" version that can
be selected during configuration.  Currently, each of these has their own
way for users to indicate this, which makes it impossible for CPAN clients
or other build tools to help users select automatically.

Going forward, the "spec" for Makefile.PL and Build.PL will include command
line options to request a "pure Perl only" build.  These will be:

* PUREPERL\_ONLY=1 (for Makefile.PL)
* --pureperl-only (for Build.PL)

These may be set in the `PERL_MM_OPT` or `PERL_MB_OPT` environment
variables just like any other command line option.

If present, distribution authors must ensure that the installed modules do
not require loading XS (whether directly or via Inline) or dynamically
generate any platform-specific code.  The installed files must be able to
run correctly if copied to another machine with the same Perl version but a
different architecture (e.g. "fatpacking" an application).  If this
condition can not be met, configuration must exit with an error.

### Environment variables for testing contexts

The Oslo Consensus defined two testing contexts: `AUTOMATED_TESTING` and
`RELEASE_TESTING`.  Of these, `AUTOMATED_TESTING` has been the most
confusing, as it sometimes was used to mean "don't interact with a user"
and sometimes "run lengthy tests".

We also (briefly) discussed how some tools like Dist::Zilla are using
`AUTHOR_TESTING` distinct from `RELEASE_TESTING`.

Distribution authors should now follow these semantics:

* `AUTOMATED_TESTING`: if true, tests are being run by an automated testing
  facility and not as part of the installation of a module; CPAN smokers
  must set this to true; CPAN clients must not set this

* `NONINTERACTIVE_TESTING`: if true, tests should not attempt to interact
  with a user; output may not be seen and prompts will not be answered

* `EXTENDED_TESTING`: if true, the user or process running tests is willing
  to run optional tests that may take extra time or resources to complete.
  Such tests must not include any development or QA tests.  Only tests of
  runtime functionality should be included.

* `RELEASE_TESTING`: if true, tests are being run as part of a release QA
  process; CPAN clients must not set this variable

* `AUTHOR_TESTING`: if true, tests are being run as part of an author's
  personal development process; such tests may or may not be run prior to
  release.  CPAN clients must not set this variable.  Distribution
  packagers (ppm, deb, rpm, etc.) should not set this variable.

There are already two libraries on CPAN to make it easier to test these
variable correctly:

* [Test::Is](http://p3rl.org/Test::Is)
* [Test::DescribeMe](http://p3rl.org/Test::DescribeMe)

These variables must be set for all phases of configuration and testing,
including running Makefile.PL or Build.PL and subsequent build and test
phases.

CPAN smokers and integration testers must indicate automated,
non-interactive testing and may request extended testing, depending on
their resources.

CPAN clients are free to request non-interactive or extended testing
depending on their needs or configuration.

CPAN smokers and clients that "must not set" a variable also must not clear
it if it is already set externally.

### Amendments to the Build.PL spec

David Golden and Leon Timmermans have been working on a
[Build.PL](https://github.com/Perl-Toolchain-Gang/cpan-api-buildpl) spec
to describe how any Perl build tool using Build.PL must behave.  It is
necessarily based on Module::Build, but does not need to follow its
behaviors exactly.

The group agreed that the use and semantics of `.modulebuildrc` should
be excluded from the specification.

### Installed distributions database

One of the QA hackathon projects was the creation of a replacement
for packlists.  An installed-distribution database would facilitate
easy inventory of installed distributions, uninstall tools and tracking of
the dependency graph of installed modules.

The group agreed that because modules can be installed into many different
locations, any such database would need to be "per @INC" and that it would
need to stack in the same way that @INC itself does.  That means that
adding paths to @INC could change what the database sees as installed.

Such a database system must not require any non-core dependencies, but
could offer enhanced capabilities if recommended CPAN modules are
installed.

Other implementation details are left to anyone designing such a system.

### Post-installation testing

Several people at the hackathon have been interested in a system for
running module tests after installation, for example to ensure that
upgraded dependencies don't break a module or to test overall integrity.

The group agreed that any such testing must make all distribution files
available during testing -- tests must be run from within a distribution
tarball directory.  Any such tests must be run using new `make` or
`Build` targets: `make test-installed` or `Build test-installed`.  These
should be equivalent to `make test` or `Build test` but without adding
`blib` to @INC.  The `prove` application must not be used.

The group also agreed that any such tests need to respect how modules can
be shadowed in @INC.  Setting PERL5LIB could change which is the
"installed" distribution and thus which tests should run.  Coordination
with an installed distribution database was encouraged.

Other implementation details, including whether the distribution directory
is saved from the initial installation or retrieved fresh from CPAN/BackPAN,
are left to anyone designing such a system.

## META file specification

### The 'provides' field

The 'provides' field of the
[CPAN::Meta::Spec](http://p3rl.org/CPAN::Meta::Spec) requires a 'file'
sub-key, but the meaning was unclear for dynamically-generated packages.
We agreed that the 'file' key must refer to the actual file within the
distribution directory that originates the package, whether that is a .pm
file or a .PL or other dynamic generator.

### Improving on 'conflicts'

We briefly discussed some of the known problems with the 'conflicts' key
within prerequisite data.

What most developers seem to want is a way to indicate that installing a
particular module is know to break other modules of particular versions.
E.g. upgrading Foo to 2.0 breaks any Bar before 3.14.

We encouraged anyone interested in improvements to prototype it using an
`x_breaks` or similar custom key and getting patches to support it into
CPAN clients.  Once battle tested, it could be a candidate for a future v3
of the spec.

## PAUSE and CPAN

### Long-term goal for distribution-level data on PAUSE

Several of the PAUSE issues discussed highlighted the need for PAUSE to
maintain not just package (namespace) level index and permission data, but
also "distribution" level data.  This would allow, for example,
transferring permissions for a distribution as a unit instead of needing
to transfer permissions on all packages.

We agreed that this is the right long-term goal, but that other proposals
would be implemented in the near-term to solve current issues.

### Case insensitive package permissions

While not discussed directly, it should be noted that PAUSE package
permissions will shortly become case-insensitive, but case-preserving
to ensure that indexed modules would be unique even if installed on a
case-insensitive file system.

### Rules for distribution naming

Many CPAN ecosystem websites and tools treat a "distribution name" as
a unique identifier, even though nothing has enforced uniqueness to date.
Allowing non-uniqueness is confusing at best and a security risk at worst.

Going forward, distributions uploaded to PAUSE must have a name that
"matches" the name of an indexed package within the distribution and the
uploader must have permissions for that package or else the entire
distribution will not be indexed.

For example, if DAGOLDEN uploads Foo-Bar-1.23.tar.gz, the distribution name
is "Foo-Bar" and there must be an indexable "Foo::Bar" package within the
distribution.

There are about 1000 distributions on CPAN that do not follow this rule and
they will be grandfathered, though they are encouraged to conform to the
standard either by renaming the distribution, adding a new .pm file or by
introducing a properly named package internally.

For example, LWP ships as libwww-perl-6.05.tar.gz.  If it included `package
libwww::perl;` into one of its .pm files, that package would be indexed and
would conform with the standard.

Technically, the correct package could also be declared only in the
META.json file using a 'provides' field.  In such a case the 'file' sub-key
must be 'META.json' to indicate that 'META.json' is the file responsible
for declaring the package.

### Flagging abandoned modules and modules requesting help

Currently, when a CPAN author passes away, his or her module permissions
are transferred to a fake author called 'ADOPTME'.  Volunteers can step
up to request a takeover if they wish to maintain them.

We agreed that in the short-term, a similar mechanism should be used to
signal abandonment or that an author is looking for someone to share
responsibility.  Unlike the case where an author is deceased, these will
use **co-maint** privileges as a signaling mechanism so that the original
author may remove them as needed.

(In the long-term, the group hopes that a distribution-level data model for
PAUSE will be able to address these needs more directly.)

CPAN search engines and other community sites may use these permissions
markers and associated meanings to communicate the status of distributions.

* **ADOPTME** as **primary**: this generally indicates a deceased author.
  Volunteers can request a takeover via modules@perl.org.

* **ADOPTME** as comaint: this indicates a verified, non-responsive author.
  The community may propose that a package be so marked following the same
  rules as for a take-over (i.e. multiple attempts to contact the author
  and a request via modules@perl.org).  Volunteers can request a takeover
  of an ADOPTME module via modules@perl.org without an additional waiting
  period.

* **HANDOFF** as comaint: this indicates that an author wishes to
  permanently give up the primary maintainer role to someone else

* **NEEDHELP** as comaint: this indicates that an author seeks people to
  help maintain the module, but plans to continue as primary maintainer

With the exception of a 'takeover' from ADOPTME (which must go through
modules@perl.org), CPAN authors must manage these comaint privileges using
the regular PAUSE interface.

An author may also voluntarily transfer primary or co-maint to ADOPTME to
indicate that PAUSE admins may transfer permissions immediately to anyone
who requests it.

### Automating PAUSE ID registration

Historically, PAUSE ID's have been manually approved, often with a
substantial delay.  We agreed that assuming appropriate protections against
bots/spam are in place, PAUSE should move to an automated approval system.
This would bring it in line with other programming language repositories
and open source community sites.

Additionally, we agreed that unused, inactive PAUSE IDs should be deleted
and made available for reuse after a period of time.  Specifically, any
PAUSE ID that ever uploaded anything must not be deleted (because the files
exists on BackPAN under that PAUSE ID).  A login to PAUSE (or via a proxy
like rt.pcan.org) is sufficient to indicate activity.  Inactive IDs will
not be deleted without a warning message about logging in to PAUSE.

### Automating CPAN directory cleanup

Approximately half the files on CPAN are older than 5 years.  Many authors
never clean up old distributions.  In order to keep the size of CPAN down,
we agreed that under certain conditions, old distribution will be
automatically scheduled for deletion (and will thereafter only exist on
BackPAN).

For a distribution to be selected for deletion, there must be at least 3
stable releases.  Anything older than the oldest of those 3 will be
scheduled for deletion if it is older than 5 years and is not indexed in
the 02packages file.

All perl tarballs will be excluded from deletion, of course.

Scheduled deletion will notify the author as usual and they will have the
usual period of time to cancel the scheduled deletion.

Cleanup will be implemented on some sort of rolling basis by author ID to
avoid bothering authors with frequent deletion notices.

### Module registration

The group agreed that the PAUSE module registration has largely outlived its
usefulness.  Because only a fraction of CPAN modules are registered,
registration does not provide a comprehensive source of metadata (e.g.
"DSLIP") and much of the information registration covers is more widely
available via META files.

The group acknowledged the remaining benefit has been that new CPAN authors
often attempt to register their first module and benefit from feedback, but
felt that other venues, such as [PrePAN](http://prepan.org/), would offer a
better new author experience.  In particular, PrePAN offers community
participation beyond one or two PAUSE admins and a wealth of examples to
learn from (without having to search through a mailing list archive).

Therefore, we agreed that existing PAUSE documentation will be changed to
direct new (and experienced) authors to PrePAN for guidance.

Soon, PAUSE will stop publishing the module registration database to CPAN
mirrors.  (The index file will exist but be empty to avoid breaking CPAN
clients that expect it.)  After an assessment period, module registration
will likely be closed and this feature will be retired from PAUSE.

## Participants in the Lancaster Consensus discussions

Discussions lasted over 3 days, participants came and went, but each day
had about 20 people.  Thank you to the following participants:

Andreas König, Barbie, Breno Oliveira, Chris Williams, Christian Walde,
David Golden, Daniel Perrett, Gordon Banner, H. Merijn Brand, James
Mastros, Jens Rehsack, Jess Robinson, Joakim Tørmoen, Kenichi Ishigaki,
Leon Timmermans, Liz Mattijsen, Matthew Horsfall, Michael Schwern, Olivier
Mengué, Paul Johnson, Peter Rabbitson, Philippe Bruhat, Piers Cawley,
Ricardo Signes, Salve J. Nilsen and Wendy van Dijk

(Apologies to anyone present who was left off the list.  Email dagolden at
cpan dot org or send a pull request to be added.)

## Copyright and License

This document is Copyright 2013 by David A. Golden and contributors.  You
can redistribute it and/or modify it under the same terms as the Perl 5
programming language system itself.
