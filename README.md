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
* Lando

## Getting Started

### Setup Identity Provider
Leverage a free trial of Okta to use as a Dev IDP:
https://developer.okta.com/docs/guides/add-an-external-idp/saml2/configure-idp-in-okta/

### Setup Service Provider
The following instructions are taken from [this quick start guide]https://simplesamlphp.org/docs/stable/simplesamlphp-sp).

1. `cp config-templates/authsources.php config/authsources.php`
2. Replace the contents of config/authsources.php with:
    ```
    $config = [

        'admin' => [
            'core:AdminPassword',
        ],

        /* This is the name of this authentication source, and will be used to access it later. */
        'default-sp' => [
            'saml:SP',
        ],
    ];
    ```
3. `mkdir cert`
4. `cd cert`
5. `openssl req -newkey rsa:3072 -new -x509 -days 3652 -nodes -out saml.crt -keyout saml.pem`
6. Replace:
    ```
    'default-sp' => [
        'saml:SP',
    ],
    ```
    With:
    ```
    'default-sp' => [
        'saml:SP',
        'privatekey' => 'saml.pem',
        'certificate' => 'saml.crt',
    ],
    ```
    In the config/authsources.php file.
7. `cd ..`
8. `cp metadata-templates/saml20-idp-remote.php metadata/saml20-idp-remote.php`
9. Replace the contents of metadata/saml20-idp-remote.php with the metadata
    provided by the IDP.

    Leverage the tool built into the admin of the SP to convert XML metadata
    to the format that simplesamlphp expects, similar to this:
    ```
    $metadata['https://idp.sunny.saml.lndo.site:[port-number]'] = [
        'SingleSignOnService'  => 'https://idp.sunny.saml.lndo.site:[port-number]/saml2/idp/SSOService.php',
        'SingleLogoutService'  => 'https://idp.sunny.saml.lndo.site:[port-number]/saml2/idp/SingleLogoutService.php',
        'certificate'          => 'example.pem',
    ];
    ```
10. Add this entry to the default-sp array in config/authsources.php, but replace
    the url with the one representing your IDP:
    ```
    'idp' => 'https://idp.sunny.saml.lndo.site:33132'
    ```
11. `cp config-templates/config.php config/config.php`
12. Set up the security configurations for accessing the admin interface for
    SimpleSAMLPHP as a Service Provider, similar to this:
    ```
    /**********************************
     | SECURITY CONFIGURATION OPTIONS |
     **********************************/

    /*
     * This is a secret salt used by SimpleSAMLphp when it needs to generate a secure hash
     * of a value. It must be changed from its default value to a secret value. The value of
     * 'secretsalt' can be any valid string of any length.
     *
     * A possible way to generate a random salt is by running the following command from a unix shell:
     * LC_CTYPE=C tr -c -d '0123456789abcdefghijklmnopqrstuvwxyz' </dev/urandom | dd bs=32 count=1 2>/dev/null;echo
     */
    'secretsalt' => 'adsfldas;fkl;asd',

    /*
     * This password must be kept secret, and modified from the default value 123.
     * This password will give access to the installation page of SimpleSAMLphp with
     * metadata listing and diagnostics pages.
     * You can also put a hash here; run "bin/pwgen.php" to generate one.
     */
    'auth.adminpassword' => 'Passw0rd',

    /*
     * Set this options to true if you want to require administrator password to access the web interface
     * or the metadata pages, respectively.
     */
    'admin.protectindexpage' => true,
    'admin.protectmetadata' => false,
    ```
13. Make sure to modify the session store information since by default both Drupal
    and SimpleSAMLPHP use the phpsession. It is best to use a sql store with a similar
    configuration as this:
    ```
    /*
     * Configure the data store for SimpleSAMLphp.
     *
     * - 'phpsession': Limited datastore, which uses the PHP session.
     * - 'memcache': Key-value datastore, based on memcache.
     * - 'sql': SQL datastore, using PDO.
     * - 'redis': Key-value datastore, based on redis.
     *
     * The default datastore is 'phpsession'.
     */
    'store.type'                    => 'sql',

    /*
     * The DSN the sql datastore should connect to.
     *
     * See http://www.php.net/manual/en/pdo.drivers.php for the various
     * syntaxes.
     */
    'store.sql.dsn'                 => 'mysql:host=database;dbname=drupal8',

    /*
     * The username and password to use when connecting to the database.
     */
    'store.sql.username' => 'drupal8',
    'store.sql.password' => 'drupal8',

    /*
     * The prefix we should use on our tables.
     */
    'store.sql.prefix' => 'SimpleSAMLphp',
    ```
