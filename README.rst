Juju Resources |badge|
-----------------------------------------------------------------

.. |badge| image:: https://travis-ci.org/juju-solutions/jujuresources.svg
    :target: https://travis-ci.org/juju-solutions/jujuresources

Juju Resources provides helpers for charms to load binary resources from
external sources, as well as tools for creating mirrors of external resources
for network-restricted deployments.

This is intended as a stop-gap until Juju has core support for resources,
as well as to prototype what features are needed.

The full documentation is available online at: http://pythonhosted.org/jujuresources/


Installing
----------

Install Juju Resources using pip:

    pip install jujuresources


Charm Usage
-----------

A charm using Juju Resources will need to define a ``resources.yaml``,
such as::

    resources:
        my_resource:
            url: http://example.com/path/to/my_resource.tgz
            hash: b377b7cccdd281bc5e4c4071f80e84a3
            hash_type: sha256
        my_py:
            pypi: jujuresources>=0.2
    optional_resources:
        my_optional_resource:
            url: http://example.com/path/to/my_optional_resource.tgz
            hash: 476881ef4012262dfc8adc645ee786c4
            hash_type: sha256

Then, once the charm has installed Juju Resources, it can fetch
and verify resources, either in Python::

    from jujuresources import fetch, verify, install, config_get

    if not fetch(mirror_url=config_get('resources_mirror')):
        print "Mandatory resources did not download; check resources_mirror option"
        sys.exit(1)
    install('my_py')

    fetch('my_optional_resource', mirror_url=config_get('resources_mirror'))
    if verify('my_optional_resource'):
        install('my_optional_resource', destination='/usr/lib/myres', skip_top_level=True)

Or via the command-line / bash::

    if ! juju-resources fetch -u `config-get resources_mirror`; then
        echo "Mandatory resources did not download; check resources_mirror option"
        exit 1
    fi
    juju-resources install my_py

    juju-resources fetch -u `config-get resources_mirror` my_optional_resource
    if juju-resources verify my_optional_resource; then
        juju-resources install my_optional_resource -D /usr/lib/myres -s
    fi


Mirroring Resources
-------------------

If you will need to deploy charms in an environment with limited network access,
you can create a mirror ahead of time, or on a gateway node which has access::

    mkdir local_mirror
    juju-resources fetch --all -d local_mirror -r http://github.com/me/my-charm/blob/master/resources.yaml
    juju-resources serve -d local_mirror

You will then have a lightweight HTTP server running to which you can set the
charm's ``resources_mirror`` (or equivalent) config option to point to,
serving all (``--all``, optional as well as required) resources defined in the
remote ``resources.yaml`` (``-r <url-or-file>``), which are cached in the
``local_mirror`` directory (``-d local_mirror``).

Note that the charms will need to be able to access the machine and port you run
the mirror on, and the charms must support a config option to point Juju Resources
to the mirror (as well as handle the possibility that their resources may not
be available when they are first deployed).
