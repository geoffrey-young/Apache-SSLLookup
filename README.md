# NAME

Apache::SSLLookup - hooks for various mod\_ssl functions

# SYNOPSIS

in httpd.conf:

    # pre-loading via PerlModule or startup.pl is REQUIRED!!!
    PerlModule Apache::SSLLookup

in any handler:

    sub handler {
      my $r = Apache::SSLLookup->new(shift);

      my $request_is_over_ssl = $r->is_https;

      my $value = $r->lookup_var('SSL_CLIENT_VERIFY');

      ...
    }

# DESCRIPTION

Apache::SSLLookup is a glue layer between Perl handlers
and the mod\_ssl public API.  under normal circumstances, you would
use `$r>subprocess_env()` to glean information about mod\_ssl.
for example,

    my $request_is_over_ssl = $r->subprocess_env('HTTPS');

however, this is only possible after mod\_ssl runs its fixups -
that is, Perl handlers can only accurately check the
`subprocess_env` table for mod\_ssl information in the
PerlResponsePhase or later.

this module allows you to query mod\_ssl directly via its public
C API at any point in the request cycle.  but without using C,
of course.

# METHODS

there are only three methods you need to be concerned with.

- new()

    to use this class you create an `Apache::SSLLookup` object.
    `Apache::SSLLookup` is a subclass of `Apache::RequestRec`
    so you can simply call `new()` and get on with your business.

        my $r = Apache::SSLLookup->new($r);

- is\_https()

    returns true if mod\_ssl considers the request to be under SSL.

        my $request_is_over_ssl = $r->is_https;

    you can call this function any time during the request, specifically
    before mod\_ssl populates `subprocess_env('HTTPS')` in the fixup
    phase.

    you must be using Apache 2.0.51 or greater for this method to
    accurately reflect the SSL status of the request.

- lookup\_var()

    returns the value of various mod\_ssl environment variables.

        my $value = $r->lookup_var('SSL_CLIENT_VERIFY');

    you can call this function any time during the request, specifically
    before mod\_ssl populates `subprocess_env()` in the fixup phase.

# NOTES

this module is for Apache 2.0 exclusively.  it will not work with
Apache 1.3.

you MUST MUST MUST preload this module with PerlModule or from
a startup.pl.  what if you don't?  the short answer is that this
module will do nothing for you.  the long answer is that unless
you preload the module it will not be able to glean the optional
function definitions from mod\_ssl.  I'm still trying to figure
out why not...

# AUTHOR

Geoffrey Young <geoff@modperlcookbook.org>

# COPYRIGHT

Copyright (c) 2004, Geoffrey Young

All rights reserved.

This module is free software.  It may be used, redistributed
and/or modified under the same terms as Perl itself.
