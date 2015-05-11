# The Berlin Consensus

At the first Perl QA Hackathon (QAH) in 2008 in Oslo, a number of QA and
toolchain authors, maintainers and experts came together to agree on some
common standards and practices.  This became known as
["The Oslo Consensus"](https://github.com/Perl-Toolchain-Gang/toolchain-site/blob/master/oslo-consensus.md).

At the 2013 QAH in Lancaster, a similar brain trust came together to
address new issues requiring consensus.  This became known as
["The Lancaster Consensus"](https://github.com/Perl-Toolchain-Gang/toolchain-site/blob/master/lancaster-consensus.md).

At the 2015 QAH in Berlin, another group assembled to address new issues,
with a particular focus on toolchain governance and recommended standards
of care for CPAN authors.

As with other consensus discussions, the speed of implementation of any
ideas discussed will depend on the interests and availability of volunteers
to do the work.

## CPAN standards of care

An ongoing challenge for Perl (and many other languages) is how to balance
the benefits of a large repository of open source libraries (i.e. CPAN)
against the fragility of large dependency trees.  The consensus discussion
group agreed on some recommended practices for CPAN authors that will, if
widely adopted, improve the general standard of care of CPAN distributions
and reduce the fragility of large CPAN dependency trees.

### The river analogy

As an analogy to guide recommendations, the group considered CPAN like a
"river".  Distributions with nothing depending on them are all the way
"downriver".  Once a distribution gains a dependent, it moves slightly
"upriver".

Distributions can have *direct* dependents — things that list it among
prerequisites — but direct dependents may have dependents, too, all the way
down to distributions with no dependents at all.

By examining this *deep dependent tree* to find the *total* number of
downriver dependents, we can assess how far "up the river" anything is,
which is tantamount to describing how many things it has the potential to
break.

As distributions have more and more total dependents in their deep
dependent trees, the further upriver they are.  All the way upriver lies
the toolchain — modules like ExtUtils::MakeMaker, which, if buggy or
broken, can break all of CPAN.

Visually, if a module upriver breaks — fails to build, fails tests, breaks
compatibility, etc. — it pollutes the river and everything downriver of it
suffers.

Therefore, a distribution's position in the river is a guide to the
recommended standards that an author should apply to its care.

As a distribution grows in popularity and gains direct dependents, those
dependents grow the deep dependent tree; the distribution moves upriver and
the standards that responsible authors should apply change accordingly.

Authors, who could have distributions at multiple places along the river,
should consider distributions individually, and avoid applying downriver
standards to distributions that have moved upriver.

The group also discussed how we never know how much of DarkPAN code depends
on a given distribution, making any CPAN-focused analysis of the river only
an approximation of the true number of downriver dependents.

### Recommended practices for CPAN authors

For the sake of discussion, the group arbitrarily divided the river into
"way downriver", "way upriver" and "in the middle", and considered
recommended practices for each.

While it wasn't discussed at the time, subsequent analysis found:

* ~50 distributions with 10,000 or more total dependents
* ~200 distributions with 1000 to 9,999 total dependents
* ~2,000 distributions with 10 to 999 total dependents
* ~8,000 distributions with 1 to 9 total dependents
* ~16,000 distributions with no known dependents

Again, while it wasn't discussed, if one needed guidance about whether a
distribution is upriver or downriver or in the middle, one might consider
the three groups roughly like this:

* "way downriver" — zero to low tens of total dependents
* "in the middle" — high tens to low thousands of total dependents
* "way upriver" — high thousands to tens of thousands of total dependents

The recommendations that follow for these groups are aspirational.  Not
every distribution can or will do everything here, but the more it can, the
better off users of that distribution will be.

#### Practices for distributions "way downriver"

Any author's first upload is, by definition, way downriver.  Brand new CPAN
authors should read a good "how to" for CPAN authorship, such as
the "About PAUSE" pages and "perldoc perlnewmod".

Downriver distributions should be "well-formed", following many of the
basic ["Kwalitee" rules](http://cpants.cpanauthors.org/kwalitee) described
on CPANTS.  They should have documentation (ideally spell-checked), a "t/"
directory with tests (that are run before shipping, e.g. with "make
disttest"), and a clearly stated license.

Distributions should respect CPAN namespace conventions.

Distributions should include a "Changes" file that highlights key
differences between releases.  They should have a META.json file that
follows the <a
href="http://p3rl.org/CPAN::Meta::Spec">CPAN::Meta::Spec</a> and a
corresponding META.yml file for older perls.

The author should decide on an "issue tracker", whether the default
rt.cpan.org or an issue tracker combined with a source code repository and
include the issue tracker URL in the META.json file.  Distribution
documentation should include contact information if it differs from the
author's cpan.org email address.

If a distribution duplicates features of existing modules, the
documentation should describe why it was created and how it differs from
other, similar modules (e.g. a "SEE ALSO" section).

There are also some practices that distributions should avoid.

When configuring, building, testing or installing: don't attempt to "phone
home" over the Internet; don't modify the filesystem outside the
distribution directory or the system temporary directory; don't send
emails.

Don't hijack other modules by installing a .pm file that overwrites or
otherwise shadows a module that ships from another distribution.

Don't run "author tests" (e.g. pod formatting, coverage, spelling, etc.)
on end-user systems.

Don't be malicious.

Don't be rude.

#### Practices for distributions "in the middle"

Distributions "in the middle" should follow all the recommendations of
those "way downriver", plus additional recommendations.

Distributions should plan for API stability.  Breaking changes should
be made as rarely as possible and should occur after a period of
deprecation.  Including a statement about a stability policy in
documentation is recommended as well, to help end-users know what to
expect.

Distributions should aim to be portable across "mainstream" operating
systems, whenever possible.  They should attempt to support older Perls
(e.g. 5.8 or 5.10) and should, regardless, have an explicit minimum perl
version in their prerequisite metadata.  Portability commitments should
also be included in documentation.

Distributions should have a public source code repository (listed in the
META files), including contribution instructions.  Repositories should be
connected to some sort of [continuous integration service](https://encrypted.google.com/search?hl=en&q=free%20hosted%20continuous%20integration)
for early identification of commits that cause tests to fail.

Distributions should be licensed under the terms of Perl itself or else a
compatible [OSI-approved license](http://opensource.org/licenses/).  E.g. a
GPL-only (or other "viral" license) may limit how far upriver a
distribution can go.  Note, also, that a "public domain" dedication is
[not always legally valid](https://en.wikipedia.org/wiki/Public_domain#Dedicating_works_to_the_public_domain)
and is not an OSI-approved license.

Distribution authors should pay attention to the issue tracker and
at least acknowledge bug reports in a timely fashion.

Distributions should have a co-maintainer registered on PAUSE or other
documented succession plan in case something happens to the original author.

Distributions should aim for high quality releases: they should have good
test coverage; authors should use author-only tests of distribution quality
before shipping (e.g. in xt); authors should monitor CPAN Testers results
to identify and fix broken releases.

Before releasing a stable, indexed version to CPAN, authors should release
a non-indexed, developer release to CPAN and monitor the [CPAN Testers
Matrix](http://matrix.cpantesters.org/) results to unsure nothing is broken
across major operating systems or versions of Perl. (N.B. typically 36 to 48
hours is sufficient for good coverage.)

Distribution authors should be mindful of a distribution's place in the
"CPAN river".  They should pay attention to the stability and quality of
upriver dependencies and should consider the stability and quality of a
distribution before adding it as a new dependency.  They should monitor the
total number of downriver dependents to reassess the standards of care to apply
to the distribution.

For distributions in the upper-middle parts of the CPAN river (e.g. 1000+
total downriver dependents), authors should consider regular testing of
some or these downriver dependents against new versions of the distribution
before shipping a stable release.

#### Practices for distributions "way upriver"

Distributions "way upriver", should follow all the recommendations of
those "in the middle" and "way downriver", plus additional
recommendations.

Distributions should undergo some sort of code review before a stable
release, so that there is more than one set of eyes responsible for the
code.

Distributions should have some documented public forum for discussing
the distribution and its evolution.  Proposed major changes should be
discussed before implementation.

Distributions should design for forward-compatibility, when possible,
so that old code can handle inputs designed for later versions of the
distribution and either adapt or throw an informative error, rather than
fail strangely.

Distributions that use C or XS should aim for C89 compatibility, the same
as Perl itself.

Distributions should do performance testing before releasing major changes
as stable.

Distributions should be tested against bleadperl, on as many platforms as
they can.

Authors should release a non-indexed development version of distributions
before an indexed, stable version for any non-trivial, non-emergency
change.

Authors should regularly check the full [CPAN testers
matrix](http://matrix.cpantesters.org/) for distributions to find failure
patterns that affect particular platforms or versions of Perl.

Authors should test at least a representative portion of direct (or even
total) downriver dependents against new versions of a distribution before
shipping a stable release.

Stable releases of distributions should not be shipped before bedtime or
the weekend or any other time when the author is unavailable to fix or
revert an unexpectedly broken release.  Authors should watch closely for
indications of breakage after release.

### Responsible forking

While we hope that authors follow standards of care consistent with a
distribution's place in the CPAN river, there will always be times when
some distribution dependency isn't meeting one's expectations.  In such a
case, a distribution always has the option to fork the dependency or to
replace it with an alternative.

Because forks can be controversial, we discussed practices for responsible
forking.

When this section talks about "forks", we intend that term to cover
literal forks (same code base to start, and same API), rewrites (new code
base, but same API) and alternative replacements (new code, with new API).

#### Before forking

Before considering a fork, authors should talk to the current maintainer(s)
about their concerns, ideally in a public forum so that concerns and
responses are the record.

Authors should also consult interested parties, such as other authors with
similar concerns (e.g. authors of other distributions who also depend on
the distribution to be forked, as well as direct downriver dependents who
might be affected by the change in their own upriver dependency tree.

#### When deciding to fork

If the original maintainer isn't able to satisfy concerns and the decision
to fork is taken, authors should notify the original maintainer(s), again
ideally in a public forum so that the history is available for future users
to review.

The author should also notify interested parties that were consulted, so
that they can participate in the process, if desired.

To help future potential users understand the differences between the
modules, the author should document the rationale for forking (focusing on
factual rather than emotional issues or personal attacks).

Authors should try not to "burn bridges".  On seeing a fork under
development, the original maintainer may reconsider his or her original
stance, or offer to hand over maintenance or otherwise find some
mutually satisfactory resolution that doesn't require the fork to be
published to CPAN.

#### After a fork is published

When the fork is shipped, the author should make a general announcement to
the same forums used for earlier discussions.

The author should carefully consider impacts on the CPAN river when
migrating.  E.g. adding a newly published fork as the dependency of a
mid-stream distribution catapults the fork upriver, which adds it as a
dependency to all downriver distributions and puts them all at risk
if the fork introduces new bugs or other surprising behavior.

When considering urging other distributions to adopt a fork in place of the
original, authors should use a "smart bomb" approach rather than a "carpet
bomb" approach.  Rather than making an appeal to all direct dependents of the
original, authors should target the appeal.  Examples of respectful
targeting include (but are not limited to):

* upriver dependencies that also depend on the original distribution (e.g.
  as part of an attempt getting the original out of the full upriver
  dependency tree)
* other, unrelated distributions that are reasonably believed to be
  affected by the same issues that prompted the fork

## Toolchain governance

The toolchain, in general terms, refers to those modules that are required
to download, build, test and install the vast majority of
distributions on CPAN.

Modules like this generally ship with the Perl core (e.g.
ExtUtils::MakeMaker) but frequently have a "dual life" independently on
CPAN.  By definition, these toolchain distributions are "way upriver" on
the CPAN river.

Other modules may not ship in the Perl core, but may be popular solutions
for specific toolchain-like tasks.  Such modules are in the "toolchain"
for distributions that rely on them.

The consensus attendees represented a sizeable cross-section of core and
popular non-core toolchain maintainers.  They agreed that toolchain
development and release practices could be improved.

They agreed to "sign on" to a "Toolchain Charter" to govern the ongoing
development of toolchain distributions.  While this discussion actually
pre-dated the CPAN river discussion, the points are largely consistent.

### Toolchain charter practices

Toolchain authors present agreed to the following principles and practices
to ensure good governance and responsible administration.

1. The toolchain has a wider scope of backwards compatibility goals than
   the Perl core, as toolchain aims to support every Perl from 5.8.1
   onwards.

2. Toolchain distributions should have more than one "primary" maintainer
   (regardless of actual PAUSE permissions) and a list should be published
   showing distributions and maintainers.

3. Functional changes to toolchain distributions need more than one set of
   eyes approving changes before shipping a stable release to CPAN

4. Major or breaking changes to a toolchain distribution should be
   discussed in a public, archived venue.  The
   [cpan-workers](http://lists.perl.org/list/cpan-workers.html) list was
   chosen as the initial venue.

5. If discussions about the evolution of toolchain distributions fail to
   achieve consensus, toolchain authors agree to defer to a designated
   "tie-breaker" authority.  The Perl pumpking (regardless of who that may
   be at any point in time) was the initial choice for tie-breaker.

6. No stable distribution should ship until some degree of stability is
   verified (e.g. though a combination of smoke reports, dependency smokes,
   etc.); the choice of specific mechanisms were left to the discretion of
   maintainers.  The group called out one exception: emergency fixes to a
   broken stable release should proceed without delay.

7. Toolchain authors agreed that when a primary maintainer steps down or
   becomes permanently unavailable, the toolchain authors as a group will
   jointly agree on a successor.  PAUSE administrators should defer to the
   consensus (or decision of the tie-breaker) for handing over PAUSE
   permissions as needed.  Any successor should agree to the practices
   described herein.

Toolchain authors not present are strongly encouraged to agree to these
practices or to hand over their toolchain maintenance responsibilities to
others who are willing to do so.

### Model behavior we want to see in other "way upriver" distributions

The group briefly discussed whether or how to urge other important,
non-toolchain, but widely depended-on distributions to sign on to the
Toolchain Charter or an equivalent.

The consensus was that the Toolchain Charter should serve as a model for
the kind of behavior we'd like to see broadly in other widely used
distributions, but that public role-modeling and public promotion was
preferred over targeted appeals or peer pressure on non-toolchain authors.

## META spec

The group discussed various potential changes to the [CPAN META
Spec](http://p3rl.org/CPAN::Meta::Spec).

### No need for META 3 yet

There was consensus that developing a "v3" META spec was not yet needed.
There was no burning platform for it, and some forward compatibility risks
to address first.  The consensus was that continuing to develop
`x_` fields was sufficient to address emerging needs.

### prereqs keys to ignore

The 'prereqs' data structure can contain fields that can't be resolved as
actual prerequisites.  Examples include "perl", "Config" and "Errno".  The
group agreed that a list of known keys should be included in the
implementation notes section of the META spec.

Specifically with regards to 'perl', we agreed that CPAN clients must never
try to install 'perl'.  If it is specified as 'requires', installation must
abort.  If specified for 'recommends' or 'suggests', CPAN clients may warn
about it, but must otherwise ignore it.

### Further clarifying 'recommends' and 'suggests'

The group agreed that these prerequisite relationships were still being
misunderstood and that the spec should be clear that "recommends" is
optional and installed by default, whereas "suggests" is optional and not
installed by default.

### CPAN client interpretation of 'x_breaks'

The group agreed that `x_breaks` needs to be implemented in CPAN clients.

`x_breaks` is a top-level key in META. It is a hash with module
name as keys and a version range that indicates the *broken* range, usually
using as a `<=` relation.  E.g. in YAML format:

    x_breaks:
        Foo::Bar: <= 1.23

When `x_breaks` is found in MYMETA or META, after successful installation,
CPAN clients should check if any of the modules listed in `x_breaks` are
installed and if their `$VERSION` matches the version range.  If so, and if
the latest version on CPAN does *not* match the `x_breaks` version range,
then the CPAN client should install the latest version of that module from
CPAN

CPAN clients may warn about unsatisfiable `x_breaks` before or after
installing a module, may prompt before installing `x_breaks`, etc.,
depending on their individual approaches to configuration, prompting and
warning.

### Need for a post-install recommendations key

There are circumstances where two distributions can each operate without
the other but both benefit from the other's presence.  In this case, each
specifying the other as 'recommended' causes a circular dependency, albeit
an optional one, which can confuse CPAN clients and dependency analysis.

To avoid this, the group agreed there needs to be a new META key that
requests that CPAN clients install a distribution after installation is
complete — not a prerequisite relationship, but an "also install"
relationship.

We agreed that Karen Etheridge volunteered to prototype such a key and discuss it
with CPAN client authors for feedback and potential implementation.

## Signaling pure-Perl

In the Lancaster Consensus, the group agreed on command-line arguments for
Makefile.PL and Build.PL to signal that a pure-Perl build is desired.  In
practice, the problem with this approach is that distribution authors must
each individually check for such flags and take them into account.

The group agreed that since there are relatively few compiler-detectors
(e.g. ExtUtils::CBuilder), it would be better to have a common environment
variable that signals to such compiler detectors to report that no compiler
is available.  This would then have an effect on all distributions using
that compiler detector.

Peter Rabbitson volunteered to pick a name for the variable and Leon
Timmermans agreed to implement it in ExtUtils::CBuilder.

## CPAN Testers grading

The historical policy of CPAN Testers was to not send failing test reports
when prerequisites could not be satisfied.  The group agreed that getting
such reports – under some other category than "FAIL" — would make it easier
to detect when upriver dependencies were broken.

Barbie and David Golden volunteered to come up with a proposal for review
on the cpan-workers list.

## PAUSE

The group considered potential changes to PAUSE administration and
policies.

### Changing adoption policies

Historically, the process for adopting an abandoned distribution allowed
the first person to petition PAUSE admins for a takeover to receive
permissions.  In light of concerns about the stability of distributions
with lots of downriver dependencies, the group thought a policy of
"first warm body" adoption was no longer a wise idea.

The group considered the opposite approach — that PAUSE admins no longer
transfer permissions at all; that if the primary maintainer were gone,
that a distribution was effectively end-of-life.  The group agreed that
this would do more harm than good.

The group agreed that for distributions with no known dependencies, the
"first warm body" rule was acceptable.  For distributions with
dependencies, PAUSE administrators will consider the number of downriver
dependencies and exercise their judgment and consider the track record and
involvement of a petitioner in the distribution in question. E.g. they will
consider existing co-maintainer status, commit involvement, etc.  Without
obvious involvement of this sort, a petitioner will need to present some
evidence of support from interested parties (e.g. direct downriver dependents).

PAUSE administrators may elect to deliberate on their decision in private,
but will announce a decision and rationale publicly.

Distributions that want to avoid succession problems are encouraged to
add co-maintainers as designated successors.

### Rewriting PAUSE not needed

The group considered briefly whether PAUSE needed to be rewritten from
scratch.  The consensus was to proceed with efforts to Plack-ify it
and use that as a basis for additional development.

### Encouraging good licensing

The group considered whether PAUSE should require a detectable license
for indexing; instead the group decided that authors should be encouraged,
but not required, to have one. E.g. the indexer email could notify authors
that no license was found and encourage them to add one.

### Responding to take-down requests

We discussed whether PAUSE should have a formal policy about take-down
requests.  The consensus was that it was unnecessary and that instead,
PAUSE admins will continue to exercise judgment.

However, we agreed that PAUSE (and all other Perl ecosystem sites) should
have clearly indicated contact information for take-down notices.

### Deleting tarballs faster

Currently, deleting a tarball on PAUSE schedules it for deletion in three
days.  We agreed that faster – nearly immediate – deletion is desirable to
get broken distributions off CPAN quickly.

However, we agreed there should be a slight delay to give backpan mirrors a
chance to catch up to store the distribution for the historical record.

Any such feature requires some sort of confirmation to help users avoid
deleting things unintentionally.
[Issue #163](https://github.com/andk/pause/issues/163) has been opened for this
feature request.

### Requiring a meta file to be indexed

The group discussed whether PAUSE should require a valid META file for a
distribution to be indexed.  When META generators start universally
providing the 'provides' field, this will get PAUSE out of the business of
guessing which packages are provided by a distribution, so encouraging
including META now will lay the groundwork for that future change.

If implemented, distributions would be required to have either META.json or
META.yml to be indexed.

The group thought this seemed reasonable, but there was some concern about
how many distributions in the recent past would have had problems if this
the policy had been in force.  Ricardo Signes volunteered to research how
many distributions uploaded recently would have had problems and review
with Andreas Koenig before implementing.

## Test-Simple roadmap

As part of the consensus discussions, the group considered the roadmap
for Test-Simple which provides the Test::Builder library upon which
Test::More and most test libraries are written.

### Problems in Test::Builder

The group identified several fundamental limitations of Test::Builder:

* Async/multi-process support is either non-existent or requires complex
  and fragile work-arounds.
* The lack of extension points has led many test libraries to invade the
  private internals of the Test::Builder singleton, or to monkey-patch its
  methods. These hacks are inherently fragile and the more the test
  ecosystem proliferates such things, the more likelihood that mixing
  arbitrary test libraries will break in unexpected ways.
* Testing test libraries requires either parsing TAP or checking against
  specific strings. This is hard, leaving many test libraries poorly
  tested, if at all. When there are tests, the tests are fragile. Overall,
  this limits the ability of test library authors to evolve TAP itself.
* More generally, having the test library so tightly coupled to TAP means
  that the capabilities of the library are limited to what TAP supports and
  there's no easy way to use the standard testing library, but output
  results in a different form (e.g. xUnit-style).
* The current Test::Builder is heavy and slow, holding too much state and
  doing to much repetitive work.

### Cost, benefits and risks

The group judged the benefit of addressing the problems above to
significant enough to justify the idea of a rewrite of the internals of
Test::Builder.

With regard to the proposed Test::Stream-based replacement, the group
agreed that the general design could address some of the problems
identified, but, in light of the risks, put forth a "punchlist" of specific
tasks to be completed before considering the specific implementation of
that design ready to move forward:

* A single Test-Simple branch with proposed code and a corresponding
  Test-Simple dev release to CPAN
* Single document describing all known issues
* Invite people to install latest dev in their daily perls for feedback
  * Write document explaining how to do so and how to roll-back
* Update CPAN delta smokes: compare test results for all of CPAN with
  latest Test-Simple stable and latest
  * On Perl versions 5.8.1 and 5.20.2; both with and without threads
  * Finding no new changes from previous list of incompatible modules doing
    unsupported things
* Line-by-line review of $Test::Builder::Level back compatibility support
* bleadperl delta smoke with verbose harness output with latest stable and
  latest dev; review line-by-line diff and find no substantive changes
  (outside Test-Simple tests themselves)
* Performance benchmarking — while specific workloads will vary, generally
  a ~15% slowdown on a "typical" workload is acceptable if it delivers the
  other desired benefits.
  * Add patches to existing benchmarking tools in Test-Simple repo
  * Run benchmarks on at least Linux and Windows

The group agreed that additional items could be added through the toolchain
governance mechanism.

## Participants in the Berlin Consensus discussions

Discussions lasted over 4 days, participants came and went, but each day
had about 20-25 people.  Thank you to the following participants:

Andreas König, Aristotle Pagaltzis, Barbie, Bulk88, Chad Granum, David
Golden, H. Merijn Brand, Helmut Wollmersdorfer, Herbert Breunung, Ingy döt
Net, Karen Etheridge, Kenichi Ishigaki, Leon Timmermans, Matthew Horsfall,
Neil Bowers, Olivier Mengué, Paul Johnson, Peter Rabbitson, Philippe
Bruhat, Ricardo Signes, Salve J. Nilsen, Slaven Rezic, Stefan Seifert,
Tatsuhiko Miyagawa, Tina Müller, and Wendy van Dijk

(Apologies to anyone present who was left off the list.  Email dagolden at
cpan dot org or send a pull request to be added.)

## Copyright and License

This document is Copyright 2015 by David A. Golden and contributors.  You
can redistribute it and/or modify it under the same terms as the Perl 5
programming language system itself.
