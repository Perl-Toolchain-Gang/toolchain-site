# The Lyon Compromise from the Perl QA Conference

Adapted from <https://gist.github.com/dagolden/9559280> to keep all the QA
conference decisions together.  Original gist was created by David Golden.

This document summarizes broad decisions about rationalizing version object
behavior based on discussions at the Lyon QAH.  Participants: David Golden,
Ricardo Signes, Karen Etheridge, Leon Timmermans, Peter Rabbitson and
Graham Knop

* version comparision should be done irrespective of the presence of
  underscores in the string used to initialize the version object

* underscore should no longer be used as a tuple separator in vstrings or
  vstring-like strings; vstrings are converted to tuples by splitting into
  *characters* (not bytes) and converting to codepoints; any elements after
  the first must be in the range 0-999

* numify/normal should produce a standarized string representation without
  underscores

* stringify should produce the best possible representation of the value
  used to initialize the version object; it should include underscores
  only if the initializing value was a non-vstring string.

* floating point numbers used as initializers are converted to a decimal
  string form at the precision limit of the architecture; people will be
  warned about this in the documentation

Examples:

    Comparison:
        - version->new(1.0203)      ==      version->new("1.0203")
        - version->new(1.02_03)     ==      version->new("1.02_03")
        - version->new(v1.2.3)      ==      version->new("v1.2.3")
        - version->new(v1.2.3_0)    ==      version->new("v1.2.3_0")

    Underscore no longer tuple separator:
        - version->new(v1.2.3_0)    ->      tuple (1,2,30)
        - version->new("v1.2.3_0")  ->      tuple (1,2,30)

    Numify/normalize don't produce underscore:
        - version->new("1.0203")->numify    -> "1.0203"
        - version->new("1.0203")->normal    -> "v1.20.300"
        - version->new("1.02_03")->numify   -> "1.0203"
        - version->new("1.02_03")->normal   -> "v1.20.300"
        - version->new("v1.2.30")->numify   -> "1.002030"
        - version->new("v1.2.30")->normal   -> "v1.2.30"
        - version->new("v1.2.3_0")->numify  -> "1.002030"
        - version->new("v1.2.3_0")->normal  -> "v1.2.30"

    Stringify should attempt to preserve string initializers:
        - version->new("1.0203")->stringify     -> "1.0203"
        - version->new("1.02_03")->stringify    -> "1.02_03"
        - version->new("v1.2.30")->stringify    -> "v1.2.30"
        - version->new("v1.2.3_0")->stringify   -> "v1.2.3_0"
        - version->new(1.0203)->stringify       -> "1.0203"
        - version->new(1.02_03)->stringify      -> "1.0203"
        - version->new(v1.2.30)->stringify      -> "v1.2.30"
        - version->new(v1.2.3_0)->stringify     -> "v1.2.30"
