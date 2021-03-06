# Puppet ``mail_server``

## Overview
Puppet ``mail_server`` is a Puppet module to enable quick deployment of a full-stack mail server using Postfix and Dovecot. It supports virtual and local mail delivery.

## Features

* Runs on Ubuntu (should also run on Debian; not yet tested)
* Single-file basic configuration of [Postfix](http://www.postfix.org/) and [Dovecot](http://www.dovecot.org/)
* Local mail delivery to users with system accounts on administrator-specified domains
* Virtual mail delivery to users without system accounts on administrator-specified domains
* Puppet 3.1+ compatible

## Getting It
When cloning, it is important to make sure that the directory name is **exactly** ``mail_server``. This is because of how Puppet handles module and class naming.

First, ``cd`` into your Puppet module path (either ``/usr/share/puppet/modules`` or ``/etc/puppet/modules`` by default on Ubuntu):
    
    cd /usr/share/puppet/modules

Then clone over HTTPS or SSH

### HTTPS
    
    git clone https://github.com/Okomokochoko/puppet-mail-server.git mail_server

### SSH
    
    git clone git@github.com:Okomokochoko/puppet-mail-server.git mail_server

## Virtual vs Local
**Local** mail delivery is mail delivered to system-local users. For instance, if you login to your server on SSH as ``joe``, and you set your server up to allow local delivery for the domains ``example.com`` and ``coffee.com``, mail sent to ``joe@example.com`` or ``joe@coffee.com`` will be delivered to ``/home/joe/Maildir``.

**Virtual** mail delivery is mail delivered to a mail store on the server, but for users who are not given system accounts. For instance, your friend Johnny is a privacy nut and doesn't want to use Gmail, but you don't want him to have shell access to your server. You can run mail for Johnny's domain ``johnny.net`` as a "virtual domain" and accept delivery of his mail. He'll be able to login via IMAP and SMTP, but you can keep his paws far away from your command line. Mail to ``johnny.net`` accounts would be stored in ``/var/mail/vmail`` by default.

Currently, virtual configuration is file-based only (using the ``hash:`` scheme in Postfix and the ``userdb passwd-file`` scheme in Dovecot). Eventually I'm hoping to expand that to support database-backed virtual configuration.

### A WARNING
You **cannot** share a domain between local and virtual delivery. Postfix will handle it one way or the other, but not both. Keep this in mind.

## A Sample Configuration:
Let's take a look at a sample configuration. We'll call it ``my-mail-server.pp``.

    class sample_mail_server {

        # Puppet resource title ``example.net`` is the local mail domain
        mail_server { 'example.net' :
            
            # The ``ssl_*`` fields should each be an absolute path to the
            # given component of the SSL process. If these do not exist,
            # PMS will create self-signed ones, which can be replaced later.
            ssl_certificate         =>  '/etc/ssl/certs/example.net.cert',
            ssl_certificate_key     =>  '/etc/ssl/private/example.net.key',
            virtual_aliases         =>  {
                'webmaster@example.com'    =>  'webmaster',
            },

            # The ``virtual_domain_users`` hash contains a key per virtual domain
            virtual_domain_users    =>  {
                'example.com'  =>  {
                    
                    # Each domain is a hash containing a key per virtual user for that domain...
                    'test'  =>  {
                        # ...and each is a hash containing settings for each user (right now only their password)
                        'password'  =>  "{SSHA256.HEX}665ec17a01855ea9e3d5fbc52727d32af02a8b67b637d7cd6f8179634f30cdaf77b7c3b5",
                    },  
                    'another'  =>  {
                        'password'  =>  "{SSHA256.HEX}665ec17a01855ea9e3d5fbc52727d32af02a8b67b637d7cd6f8179634f30cdaf77b7c3b5",
                    },  
                },  
            },  
        }
    }
    # This line is the one that actually makes our class above do something, by including it
    include sample_mail_server

To use this configuration, you'd then run:

    $ puppet apply my-mail-server.pp

Owing to the arbitrary ordering of hash entries, running multiple times may cause files to change even though you haven't changed anything.

## Generating Passwords
To generate a password, use the ``doveadm`` tool from Ubuntu's ``dovecot-imapd`` package:
    
    doveadm pw -s SHA512-CRYPT

``SHA512-CRYPT`` generates a secure SHA512 hash of the given password. There are many other schemes on Dovecot's [wiki](http://wiki2.dovecot.org/Authentication/PasswordSchemes).

## Disclaimer
This project isn't perfect. I'd like to expand its capabilities a bit in the future, but that's contingent on a variety of factors. What I'm trying to say here is that you should exercise due caution in your use of it, as you should any piece of software. If you find a bug or have a killer feature idea, please submit it to the project's GitHub Issues tracker.

## LICENSE

    Copyright 2013 Jacob Okamoto

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
