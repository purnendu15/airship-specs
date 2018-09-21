..
  This work is licensed under a Creative Commons Attribution 3.0 Unported
  License.

  http://creativecommons.org/licenses/by/3.0/legalcode

=====================================
Excel to YAML parser
=====================================

This blueprint introduces a new tool to transform site-specific information
from excel onto Pegleg manageable manifests for Airship.

Problem description
===================

During the deployment of Airship Genesis node via Pegleg, it expects that
the deployment engineer provides all the information pertained to Genesis,
Controller & Compute nodes such as PXE IPs, VLANs pertaining to Storage
network, Kubernetes network, Storage disks, Host profiles etc. as
manifests/yamls that are easily understandable by Pegleg.
Currently, these inputs are populated manually by deployment engineers based
on various unstructured sources provided to them. Considering, there would be
multiple sites for which we need to generate such data, it makes the process
cumbersome, error-prone and time-intensive.
The workaround to this problem would be to automate and standardize the site
information through a single and structured source and provide a CLI to
view and modify the site details without worrying about Pegleg structure and
conventions.

Impacted components
===================

None.

Proposed change
===============

Proposal here is to provide an Automation utility to pull the relevant
information about a specific site from standardized sources such as Formation
tool, NARAD/PINC, standardized excel spreadsheet specification. The automation
framework would be responsible to pull the site information from NARAD/PINC
(or fallback to XLS Spec whenever unavailable on NARAD/PINC)
and automatically populate the site-specific manifests that are necessary to
be consumed by Pegleg.

**Implement the Automation Engine to obtain site information from either excel
spreadsheet, Formation tool or NARAD/PINC**

   -    The Parser engine parses the necessary information from the Excel
spreadsheet via Python parsers
   -	The Parser engine outputs an intermediary yaml (as per internal format
        specification) and used within the automation engine context only.
   -	Develop and build  an internal  processor, which can consume the intermediary
yaml and generate site-specific manifests. 
   -    The site-processor uses python-jinja2 templates to generate site-specific manifests
     for baremetal, networks, profiles etc.

**Implement a CLI service**
   -	This CLI service would primarily be responsible for allowing the user
        to view and modify some of the site information before updating the
Pegleg manifests.
   -	This service would provide additional flexibility for deployment
engineers to modify/view some of the elements before deployment.

*  This blueprint covers the following
   -	Implement an Automation Engine to obtain site information from only
excel spreadsheet.
   -	Provide a CLI service to view and modify (ONLY certain attributes)
relevant site information data.

Security impact
---------------

None.

Performance impact
------------------

None.

Alternatives
------------

No existing utilities available to transform site information automatically.

Implementation
==============
* Functional Blocks
  The software consists of the following functional blocks:
  
  tugboat:

Usage
=====
* Preparation

  - Gather the following input files:

1) Excel based site specification. This file contains detail specification
covering IPMI, Public IPs, Private IPs, VLAN, Site Details etc.

2) Excel Specification to aid parsing of the above excel file. It contains
details about specific rows and columns in various sheet which contain the
necessary information to build site manifests.

3) Site specific configuration file containing additional configuration like
proxy, bgp information, interface names etc.


Assignee(s)
-----------

Primary assignee:
  Gurpreet Singh

Other contributors:
  Hemanth Nakkina
  PradeepKumar KS
  Purnendu Ghosh

Dependencies
============

None

References
==========

None
