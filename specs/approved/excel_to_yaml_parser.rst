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
  
  1)Excel Parser: It parses the excel file based on excel spec and
  generates a yaml which is further processed using site specific
  config specs and global rules to generate an intermediary yaml
  
  The site specific confi rules are passed as argument to Tugboat
  and the global design rules are constant defined by Tugboat
  
  
  2)Site Processor: The site processor consumes the intermediary yaml
  and generated site manifests based on corresponding jinja2 templates.
  
  3)Site Templates: These define the specific the formats of the manifest files
  for various entities like baremetal, network , host-profiles etc. The
  site processor applies these templates to intermediary yaml and generates
  the corresponding site manifests.
  For e.g: calico-ip-rules.yaml.j2  with generate calico-ip-rules.yaml when
  processed by the site processor.
  
  4)High-Level Workflow
  
  
  4-1) Generate Intermediary and Manifests from Site Design and Excel spec

::

  User Input                   Tugboat                 Output                    
  +---------------+        +-------------+
  | Site Design   +------> |  +------+   +------> Intermediary Yaml
  | Excel Spec    |        |  |Parser|   |
  +---------------+        |  +------+   |
                           |     |       |      
  +---------------+        |     |       |
  | Site Config   +------> |     v       |
  +---------------+        |  +---------+|
                           |  |Site     |+------> Site Manifests
  +---------------+        |  |Processor||    
  | Site Template +------> |  +---------+|
  +---------------+        +-------------+
  
  --
  4-2) Generate Manifests from Intermediary

::

  User Input                   Tugboat                 Output                    
  +---------------+        +---------------+
  | Intermediary  +------> |  +----------+ |   
  +---------------+        |  | Site     | + --------> Site Manifests
                           |  | Processor| |   
                           |  +----------+ | 
  +---------------+        |               |
  | Site Config   +------> |               |
  +---------------+        |               | 
                           |               |
  +---------------+        |               |
  | Site Template +------> |               |
  +---------------+        +---------------+
  

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
    
    4) If an Intermediary yaml file exists. In this case we do not need Excel
    and site specification
 
* Program execution
    1) CLI Options:
      -g, --generate_intermediary  Dump intermediary file from passed excel and
                                   excel spec. 
      -m, --generate_manifests     Generate manifests from the generated
                                   intermediary file
      -x, --excel PATH             Path to engineering excel file, to be passed
                                   with generate_intermediary. The -s option is
                                   mandatory with this opton.
      -s, --exel_spec PATH         Path to excel spec, to be passed with
                                   generate_intermediary. The -x option is
                                   mandatory alon with this option.
      -i, --intermediary PATH      Path to intermediary file,to be passed
                                   with generate_manifests. The -g and -x options
                                   are not required with this option.
      -d, --site_config PATH       Path to the site specific yaml file  [required]
      -l, --loglevel INTEGER       Loglevel NOTSET:0 ,DEBUG:10,    INFO:20,
                                   WARNING:30, ERROR:40, CRITICAL:50  [default:20]
      --help                       Show this message and exit.
      
     2) Example:
     
      Generate Intermediary: tugboat -g -x <DesignSpec> -s <excel spec>
     
      Generate Manifest & Intermediary: tugboat -mg -x <DesignSpec> -s <excel spec>
     
      Generate Manifest with Intermediary: tugboat -m -i <intermediary>
  

Dependencies
============

None

References
==========

None
