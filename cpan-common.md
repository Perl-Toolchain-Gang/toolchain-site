# conceptual CPAN pipeline

    $ cpan-lookup    Foo::Bar                    => JDOE/Foo-Bar-1.23.tar.gz

    $ cpan-get       JDOE/Foo-Bar-1.23.tar.gz    => /tmp/Foo-Bar-1.23.tar.gz

    $ cpan-expand    $tmp_tarball                => setup dir

    $ cpan-configure $tmp_dir                    => MYMETA.json

    $ cpan-prereqs   --missing MYMETA.json       => @modules/@dists XXX

    $ cpan-build     $tmp_dir                    => $tmp_dir

    $ cpan-test      $tmp_dir                    => $tmp_dir

    $ cpan-stage     $tmp_dir                    => $tmp_dir
    (install into $stage/phase/type/blib/... ?!)

    $ cpan-install   $phase $type
    (can't take just from XXX ?)
    things might have custom installation XXX

Stage: core vs. site vs. vendor vs. local::lib (all possible at once?)


# CPAN::Context

* hold + dispatch to objects?
* or retrieve objects?

Q: what's the necessary context needed by Distribution object

* feedback callbacks on progress
* depends how much Distributions process dependency resolution process vs. rely on external query XXX


# cpan> install Foo::Bar

1. request install for Foo::Bar                         [Queue]

2. resolve Foo::Bar to LOCAL/Foo-Bar-1.23.tar.gz        [Index]

3. Check if L/FB exists in Dist Obj Cache               [Session]

   (if so, get it + jump to 8)

4. get L/FB fron a CPAN mirror to a local cache         [Client]

5. uncompress it to a directory                         [TarZip]

6. create DistObject                                    [Dist]

7. register DistObject in cache                         [Session]

8. check config-requires                                [Dist]

   if no config-requires, jump to ...
