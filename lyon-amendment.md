# The Lyon Amendment to the Lancaster Consensus

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

New release distributions in the Perl toolchain will support the last ten
years of perl releases. They can support perl versions released even earlier, 
but they won't require a perl that has not been available for less then ten years.

As a special one-time exception, the minimum perl requirement on these
distributions will not move past v5.16.0 until July 2024, when Red Hat
Enterprise Linux and CentOS v7 leave their maintenance support phase, even
though v5.16.0 was released more than 10 years ago today.

The toolchain is generally understood, here, to include:

* CPAN
* CPAN-Meta
* ExtUtils-CBuilder
* ExtUtils-MakeMaker
* Module-Build
* Module-Metadata
* Test-Harness
* Test-Simple
* version

...and their prerequisites.

