# kerberos-devenv

[Kerberos](https://en.wikipedia.org/wiki/Kerberos_(protocol)) development environment based on [Docker Compose](https://docs.docker.com/compose/).

## Overview

The setup is composed by the following containers:
- `kerberos-server`: the Kerberos server, defined in [Dockerfile.server](Dockerfile.server), with  
configuration and the [kadm5.acl](kadm5.acl) ACL rules, which runs [krb5kdc-init.sh](krb5kdc-init.sh) script to initialize
the Kerberos database and register some principals (user and service), ahd has [start.sh](start.sh) as entrypoint.
- `kerberos-client`: the Kerberos client, defined in [Dockerfile.client](Dockerfile.client), which also relies on
[krb5.conf](krb5.conf) configuration, and has `krb5-user` packages installed, so you can use `kinit` and `klist` commands.
_It's configured to run endlessly (`tail -f /dev/null`), so you can `sh` into it to play with these commands._
- `apache-server`: the HTTP service available at `http.example.com`, based on [Apache](https://httpd.apache.org/),
defined in [Dockerfile.apache](Dockerfile.apache), which also relies on [krb5.conf](krb5.conf) configuration, and its 
service is [configured to use Kerberos auth](apache-kerberos.conf), with credentials defined in [http.keytab](http.keytab).
- `go-app`: a simple Go application, defined in [Dockerfile.app](Dockerfile.app), which also relies on [krb5.conf](krb5.conf) 
configuration, that uses [github.com/jcmturner/gokrb5](https://github.com/jcmturner/gokrb5) library to authenticate against
the Kerberos server and perform a request against the HTTP service. _It's configured to run endlessly (`tail -f /dev/null`), 
so you can `sh` into it to play with these commands._

## Usage

1. Clone this repository: `git clone https://github.com/joanlopez/kerberos-devenv.git`
2. Build the Docker images: `docker-compose build`
3. Start the Docker containers: `docker-compose up -d`

## Technical details:

- **Realm:** EXAMPLE.COM
- **Domain:** example.com
- **Example user principal:** testuser (_testpwd_)
- **HTTP service principal:** HTTP/http.example.com (_httppwd_)

## FAQs

### How can I test the Kerberos authentication?

1. Login to the Kerberos client container: `docker-compose exec kerberos-client sh`
2. Ask for a Kerberos ticket: `kinit testuser`
3. Provide the password (`testpwd`), when asked
4. Check the ticket: `klist`

### How can I register a new user principal?

1. Login to the Kerberos server container: `docker-compose exec kerberos-server sh`
2. Run: `/usr/sbin/kadmin.local -q "add_principal -pw <your-password> -kvno 1 <your-username>"`
    1. Find an example in the [krb5kdc-init.sh](krb5kdc-init.sh) script.

### How can I register a new service principal?

1. Login to the Kerberos server container: `docker-compose exec kerberos-server sh`
2. Run: `/usr/sbin/kadmin.local -q "add_principal -pw <service-password> -kvno 1 <service-principal-name>"`
    1. Find an example in the [krb5kdc-init.sh](krb5kdc-init.sh) script.
 
### How can I generate a keytab file?

1. Login to the Kerberos server container: `docker-compose exec kerberos-server sh`
2. Run: `/usr/sbin/kadmin.local -q  "ktadd -norandkey -k <keytab-output-path> <service-principal-name>@<realm>"`
   1. For instance: `/usr/sbin/kadmin.local -q  "ktadd -norandkey -k /tmp/http.keytab HTTP/http.example.com@EXAMPLE.COM"`

### How can I run the Go application?

1. Login to the Go application container: `docker-compose exec go-app sh`
2. Run the binary: `./main`
   1. It should dump out a (successful) HTTP response of the Apache server.