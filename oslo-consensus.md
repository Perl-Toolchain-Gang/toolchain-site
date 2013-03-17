# The Oslo Consensus

The Oslo Consensus was Adam Kennedy's name for an agreement reached at
the 2008 Oslo QA hackathon by the attending toolchain and QA maintainers.

The original was published on use.perl.org:
[The Oslo Consensus](http://use.perl.org/use.perl.org/_Alias/journal/36128.html)

## Context and Environment Variables

With regards to the concept of testing "contexts" we agreed that speculation
on a complete set of testing contexts is premature at this time.

However, we agreed that two particular contexts are reasonable to specify at
this time, and although any formal naming system for contexts may not exist,
we can identify how testing systems should know if these contexts apply with
the use of environment variables.

Although other shorthand methods for dealing with test contexts may be used,
the environment variables will serve as the canonical flag for when they are
enabled.

## AUTOMATED_TESTING

We agreed that the current use of the `$ENV{AUTOMATED_TESTING}` flag to
indicate an automated testing context where there is no recourse to a human
operator.

Although there was some discussion of a `$ENV{PERL_AUTOMATED_TESTING}`, the
group overwhelmingly preferred a language-agnostic flag, particularly since
the group was TAP-heavy and very much in a language-agnostic mood.

We felt that keeping the context flag language-agnostic would allow for
better reuse in the other environments where TAP-based testing would be done
with different (or more importantly mixed) languages.

Defining them as separate flags should also improve flexibility in cases
where we might want to combine contexts (for example where some
AUTOMATED_TESTING environments may want to also run the release tests, and
others not run them). It should also make implementation simpler.

## `RELEASE_TESTING`

The group felt that the use of author/release test scripts (POD testing, Perl
Critic and so on) was mature and wide-spread enough that we could readonably
define an environment variable for this context.

Different developers and systems are currently using a variety of different
flags for this context, including `$ENV{AUTHOR_TESTING}`,
`$ENV{PERL_AUTHOR_TESTING}`, `$ENV{RELEASE_TESTING}` and the use of an /xt
directory in the Perl distribution.

The group agreed that the context was best defined as "release testing"
rather than "author testing", as in by far the majority of cases the test are
most-usefully run at "make dist" time, and not continuously during
development.

Some people reported that the use of AUTHOR_TESTING had confused some
authors. Because of the use of the term "author" some authors were enabling
the flag permanently, and this was not the intended use of the context.

Because of this, the group agreed to standardize on `$ENV{RELEASE_TESTING}`
as the canonical flag for indicating that release/author tests should be run.

## The xt/ Directory

In the last few months there has been some use of a separate "xt/" directory
to contain release test scripts for a distribution, and in some cases as a
canonical way to describe release testing scripts.

We found the concept of the xt/ directory to be reasonable, primarily because
it is completely back-compatible with the legacy toolchain, and greatly
simplifies the necessary code in the test scripts (because it does not need
to have big chunks of `if ( $ENV{RELEASE_TESTING} ) { ... } elsif (
$ENV{AUTOMATED_TESTING} ) { .. }` code in it.

However, we felt that the use of a directory name as a canonical identifier
was not appropriate (despite the current use of the t/ directory) because it
would be difficult to manage, and combining contexts would become
problematic.

As a result, the group agreed that while `$ENV{RELEASE_TESTING}` would be
retained as the authoritative indicator of the release-testing context, that
support for using the "xt/" directory as a shorthand to contain release tests
should be implemented in the toolchain modules (EU:MM, M:B, M:I, etc).

This will allow for convenience in the simplest cases, and flexibility where
needed.

## META.yml release_depends Key

META.yml currently requires: (run-time dependencies), build_requires:
(build/test/install-time dependencies) and configure_requires:
(configure-time dependencies).

There was some discussion about an additional release_requires: key to deal
with the dependencies that the release tests requires. Although reasonable in
principle, there was a feeling that this would be over-specifying, and that
anyone running the release tests should be quite capable of seeing the tests
fail and installing the dependencies themselves manually.

In combined AUTOMATED_TESTING and RELEASE_TESTING contexts, the release tests
should still be smart enough to skip_all if the modules they use are not
installed.

In a similar spirit, the group recommended against specifying any additional
dependency contexts in META.yml, as this would be premature.

As a side-note, a conversation with the FreeBSD Perl maintainers later in the
day indicated that there may be some benefit in separating test_requires: out
from build_requires: as FreeBSD source packages are built by users but the
tests are NOT run, and as a result even once downstream packages start
removing build_requires: for binary packages, this still leaves FreeBSD (and
possibly Gentoo) suffering from dependency bloat in the source-build cases.

However, these issues also need further investigation. As such, the group
decided to recommend against additional dependency sections for META.yml at
this time.

## The requires: perl Key

After a quite lively discussion, it was decided that we should continue to
specify the dependency for the minimum version of Perl as a "perl: $version"
key within the requires: key. That is...

    ---
    requires:
        perl: 5.006
        Test::More: 0.42

The group decided to retain this convention rather than have some sort of
dedicated perl_version: key.

The reason for this is primarily because we can get additional value from the
current situation by extending the perl: key to the other dependency blocks.

While I'm not particularly sure why we'd want to declare an alternative Perl
version dependency for build/test context (although I'm sure someone could
come up with something), we can usefully take advantage of a perl: key in
configure_requires: as a superior alternative to running Makefile.PL, putting
a "use 5.006" key in it, and then picking apart the resulting explosion.

Because configure_requires: is authoritative, the CPAN client can check this
value and be certain that the distribution is not configurable without having
to execute the Makefile.PL.

As such, the group agreed that the optimum but non-ideal solution is extend
the perl: convention to all requires blocks.

## META_LOCAL.yml

[ed. note: subsequently, this was renamed from META_LOCAL to MYMETA]

An ongoing problem with the CPAN toolchain is with communication between the
Makefile.PL/Build.PL scripts and the CPAN client. We currently use two
entirely different communications methods and neither of them are ideal (in
fact, the Makefile.PL method could be described as hackish in the extreme).

An option that hasn't really been discussed in public forums (but has been
bounced around between some of the toolchain developers) is the idea of using
the same method we already have for communicating non-authoritative metadata
(the META.yml file) to communicate the host-specific authoritative metadata.

When we brought up the issue with the Oslo QA group, there was overwhelming
support for the idea, and near jubilation from the Debian guys, because it
would mean for the first time they can distinguish between build-time
dependencies and run-time dependencies, and thus finally remove all those
testing modules from the Debian binary package dependencies.

After some jawboning over whether to edit the existing META.yml or write a
new file (and what we might call that file) the group decided that the
toolchain modules should, in addition to the existing methods, generate a
file called META_LOCAL.yml.

This file should contain the same metadata as META.yml, but with the various
dependencies (and anything else of relevance) changed to the host-specific
values.

## Summing Up

With the above issues resolved, we felt that we were starting to exhaust both
topics that were suitably resolvable, and more importantly the members of the
group (many of whom as been stuck in TAP debates for two days).

Nonetheless, we feel that the above represents a suitably thorough set of
resolutions, and those with toolchain modules plan to start implementing the
above immediately.
