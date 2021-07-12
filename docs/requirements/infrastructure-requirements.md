# CLARIAH Requirements for Application Infrastructure

* [Introduction](#introduction)
* [Principles](#principles)
* [Definitions](#definitions)
* [Requirements](#requirements)

## Introduction

This set of requirements is largely derived from [earlier work](https://github.com/netwerk-digitaal-erfgoed/requirements-infrastructure/) done in the scope of the Netwerk Digitaal Erfgoed (NDE).

CLARIAH aims to develop a service-oriented infrastructure consisting of a wide variety of different applications, this
is often denotes as CLARIAH-as-a-Service (CLaaS) for short.

The user adoption of applications depends not only on their usefulness and quality,
but also on their reliability and performance.
When applications are slow or buggy, users will drop out.
For foundational services such as those delivered by CLARIAH, on which large groups of users depend,
this problem is even more prominent.

Infrastructure plays an important role in delivering reliable software.
This document describes requirements for a fault tolerant, performant and reliable infrastructure.
The requirements are based on modern DevOps and other best practices, including [Twelve-Factor](https://12factor.net)
and [FAIR4RS](https://www.rd-alliance.org/system/files/FAIR4RS_Principles_v0.3_RDA-RFC.pdf).
The [current NDE infrastructure](https://github.com/netwerk-digitaal-erfgoed/infrastructure) is based on these requirements and CLARIAH would do good to follow suit here.

This document is aimed at organisations and the IT operators within it that would like to take ownership of (‘borgen’) CLARIAH applications.
By joining us in following these practices,
organisations will contribute to the longevity and adoption of the applications.

For technical requirements from the perspective of the software developer/technology provider, we refer to the [CLARIAH Technical Requirements for Software and Services](software-requirements.md).

## Principles

Guiding principles of these requirements are:

### SEP

A clear separation between the application and the infrastructure it runs on.
The application code should not contain infrastructure specifics and be portable so application instances can be hosted by multiple parties, including developers that want to run or debug the application locally.
The infrastructure, in turn, should not have to change to accommodate the application.

#### Example

An infrastructure provider runs the Red Hat Linux distribution and require developers to package their applications as RPMs.
This violates the SEP principle because the artifact that is deployed is no longer portable to other, non-Red Hat, environments.

### AUTO

Processes are automated, which strengthens transparency, prevents human error and reduces the lead time between a feature request,
code changes to build that feature, and the availability of the feature to users.

#### Example

When a new application needs to be deployed, system administrators manually configure a server and copy the code to it.
When changes are made to the application code, developers have to ask administrators to deploy the latest code version.
This slows down delivery times and therefore violates the AUTO principle.
A better way would be to automatically build and deploy the application when commits are pushed to the application repository.

### REL

The infrastructure is reliable, which helps adoption of the applications.
Both software developers and end-users need reliable and performant services to get their work done.

#### Example

An infrastructure provider hosts many applications on a single virtual machine (VM).
Application usage increases, and so does server load.
To increase the VM’s resources, the system administrator takes down the VM and deploys a larger instance.
During that time the applications are unavailable to users.
This violates the REL principle.
Horizontal scaling, by running multiple application containers in parallel, would solve this issue.

## Definitions

<dl>
    <dt>Infrastructure</dt>
    <dd>The set of components that enable serving applications to users.
      Functions include not just an execution environment but also code repositories, continuous integration, deployment and monitoring.
      These functions may be delivered by a single component or be spread over multiple components (for instance, GitHub Actions as a CI pipeline, hosting on a cluster).
    </dd>
    <dt>Application</dt>
    <dd>Any software application developed by Netwerk Digitaal Erfgoed (NDE) including supporting services such as databases.</dd>
    <dt>Application repository</dt>
    <dd>The code repository that contains the application’s source code.</dd>
    <dt>Application tests</dt>
    <dd>A test suite that is included with the application.</dd>
    <dt>Application data</dt>
    <dd>Data that is produced by applications, for instance data stored in databases. Also called application state.</dd>
    <dt>Container</dt>
    <dd>A package that contains both the application and its dependencies,
        standardized by the <a href="https://opencontainers.org">Open Container Initiative (OCI)</a>.
    </dd>
    <dt>User</dt>
    <dd>
        Users of the application can be:
        <ul>
            <li>primary: persons who use the application, either directly or through a demonstrator;</li>
            <li>secondary: other software that uses the applications on behalf of the primary users. For instance: a Collection Management System that connects to the application.</li>
        </ul>
    </dd>
</dl>

## Requirements

The requirements are prioritized. Priorities follow the [MoSCoW method](https://en.wikipedia.org/wiki/MoSCoW_method):

| Priority    | Explanation                                     | Applies to                 |
| ----------- | ----------------------------------------------- | -------------------------- |
| Must have   | The requirement is necessary                    | Requirements 1 – 15  |
| Should have | The requirement is important, but not necessary | Requirements 16 – 22 |
| Could have  | The requirement is desired, but not important   | Requirement 23             |

### 1. The infrastructure runs applications that are packaged as _OCI containers_. (Must have, [SEP](#sep))

Containers are a form of lightweight virtualization, striking a good balance between isolation and performance.
Containers are self-sufficient (without external dependencies) and uniform (they look the same on the outside).
This makes them decoupled from infrastructure specifics (such as the OS used by the infrastructure provider).
The same container can be run on any Linux distribution, cloud provider (including AWS, Azure, Google and DigitalOcean)
as well as on a developer’s local Mac or Windows machine.

(This corresponds to point 12 of the [Software/Service Requirements (SR)](software-requirements.md))

### 2. The infrastructure connects to a **container registry**. (Must have, [SEP](#sep))

Built containers  (see requirement 7) are pushed to a container registry.
This can be an external registry, such as [GHCR](https://github.com/features/packages)
or one provided by the infrastructure itself.

The built artifacts must be accessible not only to the infrastructure itself but also to outside collaborators,
who can then [access and reuse](https://www.rd-alliance.org/system/files/FAIR4RS_Principles_v0.3_RDA-RFC.pdf) those same containers,
for example to run them locally (see requirement 1).

(This corresponds to point 12.1 of the [Software/Service Requirements (SR)](software-requirements.md))

### 3. The infrastructure runs _stateful applications_. (Must have)

The infrastructure can run both stateless applications (that store no state) and stateful applications, such as triple stores.
For the latter to work, the infrastructure must be able to persist data between application runs (for instance on data volumes).

(This corresponds to point 12.5 of the [Software/Service Requirements (SR)](software-requirements.md))

### 4. The infrastructure _configures applications_ through environment variables. (Must have, [SEP](#sep))

Configuration values (such as database connection strings or API URLs) must be parameterized.
The best way to do so is with [environment variables](https://12factor.net/config), a language- and OS-agnostic standard.
The infrastructure must therefore support providing these environment variables to the application.

(This corresponds to point 12.3 of the [Software/Service Requirements (SR)](software-requirements.md))

### 5. The infrastructure _captures application logs_ at stdout. (Must have, [SEP](#sep))

Traditionally, logs are written by applications to a log file or an API (such as Logstash) as dictated by the infrastructure.
This couples the application to the infrastructure it runs on.
To safeguard proper separation, the infrastructure must capture application logs at the [standard output stream](https://12factor.net/logs) (`stdout`).

(This corresponds to point 12.4 of the [Software/Service Requirements (SR)](software-requirements.md))

### 6. The infrastructure _aggregates logs_. (Must have, [REL](#rel))

To see what is going on in the running application (observability),
developers as well as operations must be able to search through logs efficiently.
This is especially relevant when multiple container instances of the application are running in parallel.
Logs can be made available through a command-line and/or a web interface.

### 7. The infrastructure _automatically builds_ the application. (Must have, [AUTO](#auto))

When commits are pushed to the application source code, the infrastructure must automatically (re)build the application,
producing an OCI container build artefact.

### 8. The infrastructure _automatically runs application tests_ when commits are pushed to the application repository. (Must have, [AUTO](#auto))

Automated tests prevent regressions only when they are run automatically on a standardized environment.

(This corresponds to point 8 of the [Software/Service Requirements (SR)](software-requirements.md))

### 9. The infrastructure _automatically delivers_ new application versions to an acceptance and/or production environment. (Must have, [AUTO](#auto))

Manual steps slow down delivery to users.
Preferably, code changes that have been committed, tested (requirements 7 and 8) and approved are delivered automatically to users at the production environment.

### 10. The infrastructure is _highly available_ corresponding to the infrastructure supplier’s SLA (Must have, [REL](#rel))

The infrastructure absorbs faults so local problems in infrastructure components do not lead to failures that users will notice.
For instance, when a server crashes or becomes unreachable, the infrastructure automatically migrates the applications to another server.
The application remains available with as little interruption as possible.

### 11. The infrastructure exposes applications at _public HTTPS web endpoints_. (Must have)

Traditionally, large parts of infrastructure are only accessible internally in the organization.
NDE applications, however, are meant for the public and therefore must be publicly accessible over HTTPS.

### 12. The infrastructure provisions and renews _TLS certificates_ for the web endpoints. (Must have, [REL](#rel))

Users must never get a ‘certificate expired’ error in their browser,
so the infrastructure must check expirations and renew certificates automatically,
for instance using [Let’s Encrypt](https://letsencrypt.org).

### 13. The infrastructure _backs up_ all application data at an agreed upon interval and has restore functionality. (Must have, [REL](#rel))

Data loss is unacceptable, especially when that data has been provided by users.

### 14. The infrastructure is _GDPR-compliant_, for instance in the way it stores data. (Must have)

If the infrastructure stores personal data, for instance logs, this must be done in a GDPR-compliant manner.

### 15. The infrastructure is _secure_. (Must have, [REL](#rel))

Access to the infrastructure must be restricted to authorized persons.
Preferably, only automated processes have access to change the infrastructure (see requirement 17).

All infrastructure and application components (such as containers) are automatically scanned for security vulnerabilities.

All software components, including the infrastructure’s OS and other packages, are continuously updated to incorporate security patches.

### 16. The infrastructure supports _zero-downtime deployments_. (Should have, [REL](#rel))

When new versions of the applications are deployed (requirement 9),
this should happen without service interruption.
Solutions include [Blue/Green](https://martinfowler.com/bliki/BlueGreenDeployment.html) or Rolling Updates.

### 17. The infrastructure configuration is _declared in code_. (Should have, [AUTO](#auto))

The infrastructure’s state is not the result of manual interventions (for example, running one-off commands),
but a reflection of the configuration as it is declared in code.
Changes made to that code are automatically rolled out,
an approach known as [Infrastructure as Code (IaC)](https://en.wikipedia.org/wiki/Infrastructure_as_code).

Having the configuration as code makes it transparent and declarative.
Preferably, the code is opened up to developers, so they can view it and propose the changes to it that are needed to run their applications.

(This relates to points 1 and 13 of the [Software/Service Requirements (SR)](software-requirements.md))

### 18. The infrastructure _captures metrics_. (Should have, [REL](#rel))

Logs (requirement 5) are used to diagnose and fix problems that have already occurred.
Metrics, on the other hand, help prevent failures.
They are values that measure the infrastructure resources and applications.
For example by measuring CPU usage, capacity can be increased in time so user service will not be interrupted.
Like log output (requirement 6), the metrics must be made available for reporting and analysis, preferably in a web interface.

### 19. The infrastructure _sends alerts_. (Should have, [AUTO](#auto))

Having a centralized place to view logs (requirement 6) is only part of a monitoring solution.
Viewing logs is pull-based, so a push mechanism for software failure notifications, on channels such as e-mail or Slack, must be added.
The infrastructure, therefore, must have a way to send out alerts based on both logs and metrics thresholds.

### 20. The application repository is _open source_. (Should have, [REL](#rel))

Collaboration between developers, both inside and outside the organisation, yields reliable software.
The ability to quickly and transparently report and fix bugs engages software developers with the application.
It is therefore best if anyone can submit issues and propose changes.

### 21. The infrastructure automatically configures _DNS_. (Should have, [AUTO](#auto))

Automating all steps for launching new applications (including registering new hostnames and configuring DNS records)
makes that process more reliable.

### 22. The infrastructure _automatically scales_ when usage changes. (Should have, [AUTO](#auto))

When usage of the application increases and decreases, the infrastructure automatically adjusts capacity within set limits.

In the case of horizontal scaling (scaling in and out) this requires load-balancing the requests to the instances.

### 23. The infrastructure has an optimized way of hosting _static files_. (Could have, [REL](#rel))

If the infrastructure is able to directly serve static files, without configuring a web server first,
this speeds up delivery.
Moreover, caching these files in a third-party Content Delivery Network (CDN) will reduce latency for users.
This is useful mainly for static HTML/JavaScript applications.
