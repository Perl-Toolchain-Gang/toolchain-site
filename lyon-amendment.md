# The Lyon Amendment to the Lancaster Consensus

🚧 This document is a *Work in Progress* and may change this week.

## Minimum-support Perl

The [Lancaster Consensus](lancaster-consensus.md) wrote:

> Going forward, the Perl toolchain will target Perl 5.8.1, released September
> 2003. This will allow toolchain modules to reliably use PerlIO and improved
> Unicode support.
>
> Because of the many Unicode bug-fixes in early 5.8 releases, toolchain
> maintainers reserve the right to later bump the minimum to 5.8.4 (which ships
> with Solaris 10).

We have revised this to:

No new release distribution in the Perl toolchain will specifiy a minimum perl
prerequisite version (whether configure, build, or test) that has not been
available for at least ten years.

As a special one-time exception, the maximum perl requirement will not move
past v5.16.0 until July 2024, when CentOS v7 leaves support, even though
v5.16.0 was released more than 10 years ago today.

The toolchain is generally understood, here, to include:
* CPAN
* ExtUtils-CBuilder
* ExtUtils-MakeMaker
* Module-Build
* Test-Harness
* Test-Simple

...and their prerequisites.

