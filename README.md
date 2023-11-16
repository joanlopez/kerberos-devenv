# kerberos-devenv

[Kerberos](https://en.wikipedia.org/wiki/Kerberos_(protocol)) development environment based on [Docker Compose](https://docs.docker.com/compose/).

## Overview

The setup is composed by the following containers:
- `kerberos-server`: the Kerberos server, defined in [Dockerfile.server](Dockerfile.server), with  
configuration and the [kadm5.acl](kadm5.acl) ACL rules, which runs [krb5kdc-init.sh](krb5kdc-init.sh) script to initialize
the Kerberos database and register some principals (user and service), ahd has [start.sh](start.sh) as entrypoint.
- `kerberos-client`: the Kerberos client, defined in [Dockerfile.client](Dockerfile.client), which also relies on
[krb5.conf](krb5.conf) configuration, and has `krb5-user` packages installed, so you can use `kinit` and `klist` commands.
It's configured to run endlessly (`tail -f /dev/null`), so you can `sh` into it to play with these commands.

## Usage

1. Clone this repository: `git clone https://github.com/joanlopez/kerberos-devenv.git`
2. Build the Docker images: `docker-compose build`
3. Start the Docker containers: `docker-compose up -d`

## Technical details:

- **Realm:** EXAMPLE.COM
- **Domain:** example.com
- **Example user:** testuser (_testpwd_)

## FAQs

### How can I test the Kerberos authentication?

1. Login to the Kerberos client container: `docker-compose exec kerberos-client sh`
2. Ask for a Kerberos ticket: `kinit testuser`
3. Provide the password (`testpwd`), when asked
4. Check the ticket: `klist`