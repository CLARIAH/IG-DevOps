# CLARIAH Technical Requirements for Software and Services

## Introduction

This document intends to specify minimal technical requirements for CLARIAH
software and software services. It is aimed at developers and external
evaluators. It provides a minimal list of specific and directly actionable criteria to hold the software
against. The aim is to ensure a certain level of software quality and
sustainability, and to lay out basic requiments needed for interoperability
within the larger CLARIAH infrastructure.

The document is loosely derived from [earlier work to establish Software Quality
Guidelines](https://github.com/CLARIAH/software-quality-guidelines/) within CLARIAH-CORE, but attempts to condense this
elaborate work to the most relevant and directly actionable points.

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”,
“SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be
interpreted as described in [RFC
2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Basic Software Requirements

### 1. The software's source code *MUST* be stored in a public **version control system** (VCS).

1. The use of a publically accessible VCS platforms such as Github, Gitlab, Bitbucket or SourceHut are *REQUIRED*. This may be hosted at an instition rather than outsourcing this to a third party.
2. CLARIAH is registered as an [organization](https://github.com/CLARIAH) on github, any CLARIAH software project *MAY* be hosted there.
3. Git is *RECOMMENDED*
4. As much as possible, such as documentation, *SHOULD* be stored alongside the code in the VCS.
5. The source code repository *SHOULD* be publically accessible from day 1 of development; it is *MUST NOT* be reduced to a release/file-transfer mechanism for completed products. Exceptions are when things are under embargo or privacy-sensitive.

### 2. A README file *MUST* be provided in the root directory of the VCS

1. This should be a readable plain-text files. Markup formats such as markdown (``README.md``, or alternatively ReStructuredText, ``README.rst``) are *recommended*. Other formats such as LaTeX, HTML, Word, etc *MUST NOT* be used for the README.
2. The README *MUST* have a clear description of what the software does (what problems it solves) and whom it is intended for.
3. The README *MUST* make the current status of the software clear, is it ready for production, experimental, or a proof-of-concept? It *MUST* also make clear whether the software is actively maintained or not. The use of the [repostatus](https://www.repostatus.org/) vocabulary is *RECOMENDED* to this end. A simple repostatus badge suffices.
4. The README *MUST* make clear who a) wrote the software and have a contact link, b) acknowledge the funders
https://github.com/IdentityPython/SATOSA5. The README *MUST* provide (or link to) installation instructions.
6. The README *MUST* provide (or link to) usage instructions for a quick start, explaining how the first task can be performed with the system.
7. The README *should* make explicit any software and minimal hardware requirements the software relies on. This includes making explicit the software's main dependencies, target OS, and minimally required resources.
8. The README *MUST* make clear (link to) a public support channel (e.g. an issue tracker, mailing list) where people can submit bug reports and feature requests.

### 3. The software *MUST* be distributed as open source under an OSI approved license

The full license *MUST* be stored in a ``LICENSE`` (``LICENSE.md``) file in the root of the VCS, the same format
recommendations and restrictions apply as to the ``README`` file.

Developers *MUST* ensure the license is not in conflict with the licenses of the dependencies they use.

* See the list of [OSI-approved licenses](https://opensource.org/licenses)

### 4. The software *MUST* be released periodically with clear version numbers

Periodic releases of the software should be released with a clear version number.

1. The use of [semantic versioning](https://semver.org/) is *strongly recommended*.
2. Each release *must* be accompanied by a tag in the VCS (e.g ``git tag``).
   This is automatically implied when you use the release mechanism by
   platforms such a GitHub, GitLab, Bitbucket.
3. Each release *must* be accompanied by release notes describing on a more
   general level what is new in the release. Again, the release mechanisms by
   major CVS platforms provide for this. Alternatively you can use a ``NEWS``
   or a ``CHANGELOG`` file.

### 5. Each release of the software *SHOULD* be installable through a proper package manager

When the software fits in a certain language ecosystem or targets a very
specific distribution/OS, it *MUST* be packaged in a package manager
fitting the language or OS. This should ensure the software *and all of its
dependencies* are installable through *one single command*:

* Python software *SHOULD* be packaged for and submitted to the [Python Package Index](https://pypi.org), installable through ``pip``.
* R software should be packaged for and submitted to [CRAN](https://cran.r-project.org/).
* Perl software *SHOULD* be packaged for and submitted to [CPAN](https://www.perl.org/cpan.html).
* Java libraries *SHOULD* be packaged for and submitted to Maven Central, installable through ``mvn``.
* Rust software *SHOULD* be packaged for and submitted to [crates.io](https://crates.io/), installable through ``cargo``.
* NodeJS software *SHOULD* be packaged for and submitted to [npm](https://www.npmjs.com/), installable through ``npm``.
* C/C++ software has no specific ecosystem for packaging. The use of
  distribution-specific packages is *RECOMMENDED*, targetting specific Linux
  distributions. The use of a standardized build system such as the autotools,
  cmake, or make is *RECOMMENDED*. Static linking *MAY* be an appropriate solution
  to handle dependecies.
* Android software *SHOULD* be submitted to [Google Play](https://play.google.com) and alternatively also [F-Droid](https://f-droid.org/).
* iOS sofware *SHOULD* be submitted to the App Store.

This step may combine both the building of software (compilation) as well the
installation. This rule shall only be deviated from if there is no suitable
packaging ecosystem for the software (C/C++) or if the software is not a fit
for the packaging ecosystem, such as the software being too high-level (e.g.  a
complex web application) and consisting of too many components for any
packaging ecosystem. In such a case, the individual components should still be
packaged as much as possible.

### 6. The software *SHOULD* have a public support channel

This can be an issue tracker as automatically provided by the major VCS
platforms, or even a simple mailing-list, on the precondition that a public
archive is available.

Users can turn to this place for reporting any bugs they find in the software,
or optionally post requests for new features. The public nature ensures that a
public knowledge base is constructed where users can find answers to earlier
posted questions, aleviating the burden on both the users as well as the
developers. The public support channel also gives an indication of community
interest in the project.

The only acceptable deviation to this rule is when the software is explicitly unmaintained
and unsupported, which must be clearly indicated in the README (see 1.3).

### 7. Software *SHOULD* be reusable

1. To foster reusability, any meaningful reusable component of your sofware *SHOULD* be
   split into reusable software libraries/tools rather than be part of an
   indivisible monolithical whole. This ensures the work can be reused where
   appropriate.
2. Software *MUST* separate code from configuration, they *MUST NOT* be hard-coded to only work on a single system or
   for a specific user; hard-coded user/installation-specific paths and sensitive credentials *MUST NOT* be part of the
   software source code. Software *MUST* be made configurable (through configuration files, command line parameters,
   environment variables) to be deployable in multiple environments.

### 8. Software *SHOULD* come with automated tests

1. To ensure software quality, software *SHOULD* have a test set consisting of
unit and/or integration tests with a fair degree of coverage.
2. Automatic *Continuous Integration* infrastructure such as GitHub Actions, GitLab CI/CD,
Jenkins or others *SHOULD* be used to automatically test the software upon any
code change.
3. Additionaly, code coverage tests and dependency version checks
*MAY* be used.

Deviations from this are only acceptable for initial proof-of-concept or highly
experimental software (see 1.3).

### 9. Software **MUST** define codemeta software metadata along with the source code

1. The VCS repository must have a ``codemeta.json`` file in the root directory containing basic high-level
   software metadata. We adopt the [CodeMeta project](https://codemeta.github.io/) to this end. The ``codemeta.json`` file is a JSON-LD file that is machine-parseable and may be
   used by metadata harvesters, for inclusion in e.g. Ineo. Moreover, it enable citability of the
   software. The codemeta file can be in part automatically generated from the metadata you already
   provided for your packages (see point 5)

### 10. Software **SHOULD** be documented.

Whereas minimal documentation is already *REQUIRED* per point 2, more extensive documentation is *RECOMMENDED*. The sources of
which *SHOULD* be stored alongside the source code, ensuring they describe the same version, and the resulting
documentation *SHOULD* be served on a website (platforms like readthedocs.io *MAY* be a good solution here).

1. When the software is a library, an API reference *MUST* be provided.
2. When the software is a webservice, a WebAPI reference *MUST* be provided.
3. When the software has a command line interface, usage example should be given in the documentation. Furthermore a ``--help`` parameter *MUST* be provided in the software itself.
4. When the software has a graphical/web user interface, the documentation *SHOULD* explain how to use it.
5. The use of API documentation as integral part of the code and tools such as doxygen, sphinx is *RECOMMENDED*.
6. The use of API documentation as integral part of the code and tools such as doxygen, sphinx is *RECOMMENDED*.

Deviations to this rule are for unsupported/proof-of-concept/experimental software, which must be clearly indicated in
the README (see 1.3).

## Software-as-a-Service Requirements (CLaaS)

### 11. Services *SHOULD* provide a simple RESTful API

A simple RESTful API is *RECOMMENDED* over more complex solutions such as SOAP. SOAP *SHOULD NOT* be used if it can be
avoided. XML-RPC *MAY* be used. CLAM *MAY* be used as a home-grown CLARIAH solution to deliver RESTful webservices around
existing tools.

### 12. Services *MUST* be packaged as containers

All software services *MUST* be packaged as [OCI](https://opencontainers.org/) containers (e.g. Docker containers).
Containers are self-sufficient (without external dependencies) and uniform
(they look the same on the outside). This makes them decoupled from
infrastructure specifics (such as the OS used by the infrastructure provider).
The same container can be run on any Linux distribution, cloud provider
(including AWS, Azure, Google and DigitalOcean) as well as on a developer’s
local Mac or Windows machine.

(This corresponds to [point 1](infrastructure-requirements.md#1-the-infrastructure-runs-applications-that-are-packaged-as-oci-containers-must-have-sep) of the [Infrastructure Requirements (IR)](infrastructure-requirements.md))

1. The container images *MUST* be published in CLARIAH's container registry at each release. This can be automated (see
   [IR7](infrastructure-requirements.md#7-the-infrastructure-automatically-builds-the-application-must-have-auto)).
2. Containers *SHOULD* be kept as lightweight application containers, not including unnecessary runtime components.
3. Configuration values (such as database connection strings or API URLs) that may vary between deployments
   *SHOULD* be parameters to the container. This is implemented through environment variables. The infrastructure in
   turn configures applications through environment variables (see [IR4](infrastructure-requirements.md#4-the-infrastructure-configures-applications-through-environment-variables-must-have-sep) and also related to point 7).
4. Containers should output all log information to ``stdout`` so it can be captured by the infrastructure (see IR5 and
   IR6).
5. Application data (state) that needs to be persistant between runs *MUST* be stored separate from the container (e.g. in a
   mounted volume) (See point 3 of the [Infrastructure Requirements (IR)](infrastructure-requirements.md))

### 13. Service developers *SHOULD* provide an initial template when of multi-container orchestration is needed

If a complex service consists of multiple interacting containers, the developers *SHOULD* provide an initial template in
the form of a Docker Compose configuration or a Kubernetes deployment configuration that illustrates how the containers
are orchestrated to form the application. This *MUST* be maintained in a VCS repository (infrastructure as code
principle). Infrastructure operators can build on this example to deploy the application.

### 14. Services *MUST* be compatible with CLARIAH's authentication and authorization infrastructure

All services open to end-users and which require some form of user authentication should be compatible with
CLARIAH's authentication and authorization infrastructure. That is, they should be able to communicate with CLARIAH's
[SATOSA](https://github.com/IdentityPython/SATOSA) Authentication Provider. It is *RECOMMENDED* to use OpenID Connect
for this communication. Instruction can be found [(TODO add link!)]().

### 15. Services *MUST* expose a public endpoint providing their specification

1. Swagger, WADL, and CLAM are *SUGGESTED* as possible interface description languages.
2. The endpoint *SHOULD NOT* be hindered by any authentication barriers.

### 16. Services *SHOULD* expose a public endpoint providing high-level codemeta metadata

This is the corollary of point 9 for software. The codemeta metadata for the service as a whole must be available at a
public endpoint returning a codemeta JSON-LD file. Note that this is not does describe the API (unlike point 14). This
is used for automatic harvesting and inclusion in e.g. Ineo.

The only exception to this rule is if the endpoint offered in point 14 already provides enough information to *automatically*
derive a codemeta representation from.

### 17. Services *MAY* participate in the CLARIN switchboard

The CLARIN switchboard aims to direct a user with a given data file to useful services. Participation is encouraged.


## Glossary


* **CLaaS** - CLARIAH as a Service
    vocabularies.
* **CLAM** - A framework developed in CLARIN/CLARIAH for building RESTful webservices with an additional generic user-interface for human end-users.
* **CodeMeta** - A third-party initiative to describe research software in Linked Open Data, and link to existing
* **Ineo** - The official portal to CLARIAH tools/services and data resources
* **VCS** - A **version control system**, e.g. Git, Mercurial, SVN, etc



