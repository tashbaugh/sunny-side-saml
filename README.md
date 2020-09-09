# sunny-side-saml

# SimpleSAMLPHP for the Rest of Us...
This project was created to serve as a companion to a presentation given by Tyler Ashbaugh.
## Purpose of Project
> Setting up the components necessary in a development environment to begin work on a SAML project is a formidable task to say the least. This project aims to be a turn key solution for developers so they can hit the ground running.
---
### Parts of the Project
This project contains three major pieces:

1. The identity provider - source of truth when it comes to authentication and profile information.
2. The service provider - the authentication middleware standing between Drupal and the Identity Provider.
3. The Drupal site - The application we wish to enable a SAML handshake in.
---
## Prerequisites
This project makes a few assumptions about developers that use it:

* Familiarity with git
* Familiarity with composer-based workflows

This projects also expects a few things are available in your local environment:

* Git
* Composer
* Lando or DDEV

## Getting Started


