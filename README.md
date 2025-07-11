# ![DNSViz](doc/images/logo-220x100.png)


## Table of Contents
* [Installation](#installation)
* [Usage](#usage)
* [Pre-Deployment DNS Testing](#pre-deployment-dns-testing)
* [Docker Container](#docker-container)


## Description

DNSViz is a tool suite for analysis and visualization of the Domain Name System
(DNS) behavior, including its security extensions (DNSSEC).  This tool suite
powers the Web-based analysis available at https://dnsviz.net/


## Installation

DNSViz is available in package repositories for popular operating systems, such
as Debian, Ubuntu, Fedora, Gentoo, and FreeBSD.  It is also available in the
Extra Packages for Linux (EPEL) repository for Red Hat Enterprise Linux
(RHEL) 8 and 9 and CentOS 8 and 9. (See
[notes](#rhel-89-or-centos-stream-89-notes) for installation on RHEL and
Centos.)  In each case, it can be installed using the package installation
commands typical for that operating system.  DNSViz can also be installed on
Mac OS X using Homebrew or MacPorts.

The remainder of this section covers other methods of installation, including a
list of [dependencies](#dependencies), installation to a
[virtual environment](#installation-in-a-virtual-environment), and
[notes for installing on RHEL 8 or 9 or CentOS Stream 8 or 9,](#rhel-89-or-centos-stream-89-notes)).

Instructions for running in a Docker container are also available
[later in this document](#docker-container).


### Dependencies

* Python (2.7, 3.5 - 3.12) - https://www.python.org/
  (Note that Python 2.7 support will be removed in a future release.)

* dnspython (1.13.0 or later) - https://www.dnspython.org/

* pygraphviz (1.3 or later) - https://pygraphviz.github.io/

* cryptography (36.0.0 or later) - https://cryptography.io/

Note that earlier versions of the software listed above might also work with
DNSViz, but are not supported.  For example, versions of cryptography as early
as 2.6 seem to work.  Also note that while DNSViz itself still works with
Python 2.7, some versions of its software dependencies have moved on:
pygraphviz 1.6 and dnspython 2.0.0 dropped support for Python 2.7.


### Optional Software

* OpenSSL GOST Engine - https://github.com/gost-engine/engine

  With OpenSSL version 1.1.0 and later, the OpenSSL GOST Engine is necessary to
  validate DNSSEC signatures with algorithm 12 (GOST R 34.10-2001) and create
  digests of type 3 (GOST R 34.11-94).  M2Crypto is also needed for GOST support.

* M2Crypto - https://gitlab.com/m2crypto/m2crypto

  While almost all of the cryptographic support for DNSViz is handled with the
  cryptography Python module, support for algorithm 12 (GOST R 34.10-2001)
  digest type 3 (GOST R 34.11-94) requires the OpenSSL GOST Engine.  That engine
  must be loaded dynamically, and there is no support for that with
  cryptography.  Thus, if you need to support algorithm 12 or digest type 3,
  you must also install M2Crypto.

* ISC BIND - https://www.isc.org/bind/

  When using DNSViz for [pre-deployment testing](#pre-deployment-dns-testing)
  by specifying zone files and/or alternate delegation information on the
  command line (i.e., with `-N`, `-x`, or `-D`), `named(8)` is invoked to serve
  one or more zones.  ISC BIND is only needed in this case, and `named(8)` does
  not need to be running (i.e., as a server).

  Note that default AppArmor policies for Debian are known to cause issues when
  invoking `named(8)` from DNSViz for pre-deployment testing.  AppArmor can be
  temporarily disabled for `named(8)` with the following:

  ```
  $ sudo apparmor_parser -R /etc/apparmor.d/usr.sbin.named
  ```

  After pre-deployment testing is finished, AppArmor for `named(8)` can be
  re-enabled with the following:

  ```
  $ sudo apparmor_parser /etc/apparmor.d/usr.sbin.named
  ```


### Installation in a Virtual Environment

To install DNSViz to a virtual environment, first create and activate a virtual
environment, and install the dependencies:
```
$ virtualenv ~/myenv
$ source ~/myenv/bin/activate
(myenv) $ pip install -r requirements.txt
```
Note that this installs the dependencies that are Python packages, but some of
these packages have non-Python dependencies, such as Graphviz (required for
pygraphviz) and OpenSSL (required for cryptography), that are not installed
automatically.

Next download and install DNSViz from the Python Package Index (PyPI):
```
(myenv) $ pip install dnsviz
```
or locally, from a downloaded or cloned copy of DNSViz:
```
(myenv) $ pip install .
```


### RHEL 8/9 or CentOS Stream 8/9 Notes

DNSViz can be installed on RHEL 8 or 9 or CentOS Stream 8 or 9 from the EPEL
repository.  Follow the instructions in this section to enable EPEL.

*RHEL 8 and 9 only*: Enable CodeReady Linux Builder and Extra Packages for
Enterprise Linux (EPEL) with following:

```
$ sudo subscription-manager repos --enable codeready-builder-for-rhel-$(vers)-$(arch)-rpms
$ sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(vers).noarch.rpm
```
(where `$(vers)` refers to version, either `8` or `9`, and `$(arch)` refers to
the architecture, e.g., `x86_64` or `aarch64`. If you are unsure, run
`sudo subscription-manager repos --list` to show available options.)

*CentOS Stream 8 or 9 only*: Enable PowerTools or CodeReady Linux Builder and
EPEL with the following:

```
$ sudo dnf config-manager --set-enabled $(tool)
$ sudo dnf install epel-release
```
(where `$(tool)` refers to the tool, either `powertools` for CentOS Stream 8 or
`crb` for CentOS Stream 9.)

For both RHEL 8 or 9 *and* CentOS Stream 8 or 9, once EPEL is enabled, install
DNSViz using `dnf`:

```
$ sudo dnf install dnsviz
```


## Usage

DNSViz is invoked using the `dnsviz` command-line utility.  `dnsviz` itself
uses several subcommands: `probe`, `grok`, `graph`, `print`, and `query`.  See
the man pages associated with each subcommand, in the form of
"dnsviz-<subcommand> (1)" (e.g., "man dnsviz-probe") for more detailed
documentation and usage.

### dnsviz probe

`dnsviz probe` takes one or more domain names as input and performs a series of
queries to either recursive (default) or authoritative DNS servers, the results
of which are serialized into JSON format.


#### Examples

Analyze the domain name example.com using your configured DNS resolvers (i.e.,
in `/etc/resolv.conf`) and store the queries and responses in the file named
"example.com.json":
```
$ dnsviz probe example.com > example.com.json
```

Same thing:
```
$ dnsviz probe -o example.com.json example.com
```

Analyze the domain name example.com by querying its authoritative servers
directly:
```
$ dnsviz probe -A -o example.com.json example.com
```

Analyze the domain name example.com by querying explicitly-defined
authoritative servers, rather than learning the servers through referrals from
The IANA root servers:
```
$ dnsviz probe -A \
  -x example.com:a.iana-servers.org=199.43.132.53,a.iana-servers.org=[2001:500:8c::53] \
  -x example.com:b.iana-servers.org=199.43.133.53,b.iana-servers.org=[2001:500:8d::53] \
  -o example.com.json example.com
```

Same, but have `dnsviz probe` resolve the names:
```
$ dnsviz probe -A \
  -x example.com:a.iana-servers.org,b.iana-servers.org \
  -o example.com.json example.com
```

Analyze the domain name example.com and its entire ancestry by querying
authoritative servers and following delegations, starting at the root:
```
$ dnsviz probe -A-a. -o example.com.json example.com
```

Analyze multiple names in parallel (four threads) using explicit recursive
resolvers (replace *192.0.1.2* and *2001:db8::1* with legitimate resolver
addresses):
```
$ dnsviz probe -s 192.0.2.1,[2001:db8::1] -t 4 -o multiple.json \
  example.com sandia.gov verisignlabs.com dnsviz.net
```


### dnsviz grok

`dnsviz grok` takes serialized query results in JSON format (i.e., output from
`dnsviz probe`) as input and assesses specified domain names based on their
corresponding content in the input.  The output is also serialized into JSON
format.


#### Examples

Process the query/response output produced by `dnsviz probe`, and store the
serialized results in a file named "example.com-chk.json":
```
$ dnsviz grok < example.com.json > example.com-chk.json
```

Same thing:
```
$ dnsviz grok -r example.com.json -o example.com-chk.json example.com
```

Show only info-level information: descriptions, statuses, warnings, and errors:
```
$ dnsviz grok -l info -r example.com.json -o example.com-chk.json
```

Show descriptions only if there are related warnings or errors:
```
$ dnsviz grok -l warning -r example.com.json -o example.com-chk.json
```

Show descriptions only if there are related errors:
```
$ dnsviz grok -l error -r example.com.json -o example.com-chk.json
```

Use the root key as a DNSSEC trust anchor to additionally indicate
authentication status of responses:
```
$ dig +noall +answer . dnskey | awk '$5 % 2 { print $0 }' > tk.txt
$ dnsviz grok -l info -t tk.txt -r example.com.json -o example.com-chk.json
```

Pipe `dnsviz probe` output directly to `dnsviz grok`:
```
$ dnsviz probe example.com | \
      dnsviz grok -l info -o example.com-chk.json
```

Same thing, but save the raw output (for re-use) along the way:
```
$ dnsviz probe example.com | tee example.com.json | \
      dnsviz grok -l info -o example.com-chk.json
```

Assess multiple names at once with error level:
```
$ dnsviz grok -l error -r multiple.json -o example.com-chk.json
```


### dnsviz graph

`dnsviz graph` takes serialized query results in JSON format (i.e., output from
`dnsviz probe`) as input and assesses specified domain names based on their
corresponding content in the input.  The output is an image file, a `dot`
(directed graph) file, or an HTML file, depending on the options passed.


#### Examples

Process the query/response output produced by `dnsviz probe`, and produce a
graph visually representing the results in a PNG file named "example.com.png".
```
$ dnsviz graph -Tpng < example.com.json > example.com.png
```

Same thing:
```
$ dnsviz graph -Tpng -o example.com.png example.com < example.com.json
```

Same thing, but produce an interactive HTML format:
Interactive HTML output in a file named "example.com.html":
```
$ dnsviz graph -Thtml < example.com.json > example.com.html
```

Same thing (filename is derived from domain name and output format):
```
$ dnsviz graph -Thtml -O -r example.com.json
```

Use alternate DNSSEC trust anchor:
```
$ dig +noall +answer example.com dnskey | awk '$5 % 2 { print $0 }' > tk.txt
$ dnsviz graph -Thtml -O -r example.com.json -t tk.txt
```

Pipe `dnsviz probe` output directly to `dnsviz graph`:
```
$ dnsviz probe example.com | \
      dnsviz graph -Thtml -O
```

Same thing, but save the raw output (for re-use) along the way:
```
$ dnsviz probe example.com | tee example.com.json | \
      dnsviz graph -Thtml -O
```

Process analysis of multiple domain names, creating an image for each name
processed:
```
$ dnsviz graph -Thtml -O -r multiple.json
```

Process analysis of multiple domain names, creating a single image for all
names.
```
$ dnsviz graph -Thtml -r multiple.json > multiple.html
```


### dnsviz print

`dnsviz print` takes serialized query results in JSON format (i.e., output from
`dnsviz probe`) as input and assesses specified domain names based on their
corresponding content in the input.  The output is a textual output suitable for
file or terminal display.


#### Examples

Process the query/response output produced by `dnsviz probe`, and output the
results to the terminal:
```
$ dnsviz print < example.com.json
```

Use alternate DNSSEC trust anchor:
```
$ dig +noall +answer example.com dnskey | awk '$5 % 2 { print $0 }' > tk.txt
$ dnsviz print -r example.com.json -t tk.txt
```

Pipe `dnsviz probe` output directly to `dnsviz print`:
```
$ dnsviz probe example.com | \
      dnsviz print
```

Same thing, but save the raw output (for re-use) along the way:
```
$ dnsviz probe example.com | tee example.com.json | \
      dnsviz print
```


### dnsviz query

`dnsviz query` is a wrapper that couples the functionality of `dnsviz probe`
and `dnsviz print` into a tool with minimal dig-like usage, used to make
analysis queries and return the textual output to the terminal or file output in
One go.


#### Examples

Analyze the domain name example.com using the first of your configured DNS
resolvers (i.e., in `/etc/resolv.conf`):
```
$ dnsviz query example.com
```

Same, but specify an alternate trust anchor:
```
$ dnsviz query +trusted-key=tk.txt example.com
```

Analyze example.com through the recursive resolver at 192.0.2.1:
```
$ dnsviz query @192.0.2.1 +trusted-key=tk.txt example.com
```


## Pre-Deployment DNS Testing

The examples in this section demonstrate the usage of DNSViz for pre-deployment
testing.


### Pre-Delegation Testing

The following examples involve issuing diagnostic queries for a zone before it
is ever delegated.

Issue queries against a zone file on the local system (`example.com.zone`).
`named(8)` is invoked to serve the file locally:
```
$ dnsviz probe -A x example.com: + example.com. zone example.com
```
(Note the use of "+", which designates that the parent servers should not be
queried for DS records.)

Issue queries to a server that is serving the zone:
```
$ dnsviz probe -A x example.com.com+:192.0.2.1 example.com
```
(Note that this server doesn't need to be a server in the NS RRset for
example.com.)

Issue queries to the servers in the authoritative NS RRset, specified by name
and/or address:
```
$ dnsviz probe -A \
      -x example.com+:ns1.example.com=192.0.2.1 \
      -x example.com+:ns2.example.com=192.0.2.1,ns2.example.com=[2001:db8::1] \
      example.com
```

Specify the names and addresses corresponding to the future delegation NS
records and (as appropriate) A/AAAA glue records in the parent zone (com):
```
$ dnsviz probe -A \
      -N example.com:ns1.example.com=192.0.2.1 \
      -N example.com:ns2.example.com=192.0.2.1,ns2.example.com=[2001:db8::1] \
      example.com
```

Also supply future DS records:
```
$ dnsviz probe -A \
      -N example.com:ns1.example.com=192.0.2.1 \
      -N example.com:ns2.example.com=192.0.2.1,ns2.example.com=[2001:db8::1] \
      -D example.com:dsset-example.com. \
      example.com
```


### Pre-Deployment Testing of Authoritative Zone Changes

The following examples involve issuing diagnostic queries for a delegated zone
before changes are deployed.

Issue diagnostic queries for a new zone file that has been created but not yet
been deployed (i.e., with changes to DNSKEY or other records):
```
$ dnsviz probe -A x example.com:example.com.zone example.com
```
(Note the absence of "+", which designates that the parent servers will be
queried for DS records.)

Issue queries to a server that is serving the new version of the zone:
```
$ dnsviz probe -A x example.com:192.0.2.1 example.com
```
(Note that this server doesn't need to be a server in the NS RRset for
example.com.)


### Pre-Deployment Testing of Delegation Changes

The following examples involve issuing diagnostic queries for a delegated zone
before changes are deployed to the delegation, glue, or DS records for that
zone.

Specify the names and addresses corresponding to the new delegation NS records
and (as appropriate) A/AAAA glue records in the parent zone (com):
```
$ dnsviz probe -A \
      -N example.com:ns1.example.com=192.0.2.1 \
      -N example.com:ns2.example.com=192.0.2.1,ns2.example.com=[2001:db8::1] \
      example.com
```

Also supply the replacement DS records:
```
$ dnsviz probe -A \
      -N example.com:ns1.example.com=192.0.2.1 \
      -N example.com:ns2.example.com=192.0.2.1,ns2.example.com=[2001:db8::1] \
      -D example.com:dsset-example.com. \
      example.com
```


## Docker Container

A ready-to-use docker container is available for use.

```
docker pull dnsviz/dnsviz
```

This section only covers Docker-related examples. For more information, see the
[Usage](#usage) section.


### Simple Usage

```
$ docker run dnsviz/dnsviz help
$ docker run dnsviz/dnsviz query example.com
```


### Working with Files

It might be useful to mount a local working directory into the container,
especially when combining multiple commands or working with zone files.

```
$ docker run -v "$PWD:/data:rw" dnsviz/dnsviz probe dnsviz.net > probe.json
$ docker run -v "$PWD:/data:rw" dnsviz/dnsviz graph -r probe.json -T png -O
```


### Using a Host Network

When running authoritative queries, a host network is recommended.

```
$ docker run-- network host dnsviz/dnsviz probe -4 -A example.com > example.json
```

Otherwise, you're likely to encounter the following error:
`dnsviz.query.SourceAddressBindError: Unable to bind to local address (EADDRNOTAVAIL)`


### Interactive Mode

When performing complex analyses, where you need to combine multiple DNSViz
commands, use bash redirection, etc., it might be useful to run the container
interactively:

```
$ docker run --network host -v "$PWD:/data:rw" --entrypoint /bin/sh -ti dnsviz/dnsviz
/data # dnsviz --help
```
