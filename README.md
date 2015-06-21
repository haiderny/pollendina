# Pollendina
Pollendina is an X.509 identity provisioning service designed to simplify mutual TLS authentication for microservices deployed in containers.

**Pollendina accomplishes this without distribution or centralized management of secrets!**

## Architecture
![Pollendina PKI infrastructure and architecture](https://raw.github.com/allingeek/pollendina/master/docs/PollendinaFlow2.png)

Provisioning a service for use in a new container would consist of the following steps:
* Tell Pollendina that a container for service X at hostname Y is about to be provisioned.
* Provision the container.
* On service initialization, create a new RSA key pair and certificate signing request.
* Call Pollendina with the CSR.
* Pollendina validates that the CSR is approved for provisioning.
* Pollendina signs the CSR with the organization's CA private key and returns the PEM encoded public key for the service in the requesting container (X.509 subject).
* The calling container installs the returned certificate and private key (either keep it in memory or write it encrypted to a volume).

## Launch Pollendina

`docker run -d -p 33004:33004 -v "$PWD":/opt/pollendina pollendina/pollendina`

The above command will start Pollendina in a new container and provision a new CA in the present working directory. The files created at PWD represent the full state of the CA. If Pollendina is stopped, and another Pollendina container is started from this folder it will resume the state of the previous CA. One file named openssl-ca.cnf is created. You can customize the settings for your CA by modifying this file and restarting Pollendina.

## Main Contributors 

  - [Jeff Nickoloff](https://github.com/allingeek)
  - [Dário Nascimento](https://github.com/dnascimento)
  - [Jason Huddleston](https://github.com/huddlesj)
  - [Madhuri Yechuri](https://github.com/myechuri)
  - [Henry Kendall](https://github.com/hskendall)

## API Guide

```
  POST /v1/authorize HTTP/1.1

  cn=<client common name>&token=<one-time-token>
```

```
  PUT /v1/sign/<one-time-token>
  
  *include CSR in PUT body*
```

  Pollendina CA can be used / tested standalone, without a client container, using ``curl`` client:

  `curl --data "cn=dario&token=100" http://192.168.59.103:33004/v1/authorize`

  `curl -v http://192.168.59.103:33004/v1/sign/100 --upload-file id.csr`

## Resources

There is a great answer on StackOverflow that goes over running a certificate authority, and creating certificates at: http://stackoverflow.com/questions/21297139/how-do-you-sign-certificate-signing-request-with-your-certification-authority

## Further Development

1. Move client initialization out of init script and into a container provisioning hook
2. Remove any client dependency on OpenSSL
3. Move to golang native SSL in Pollendina service
4. Implement persistent CSR authorization database
5. CA configuration and state to distributed store to scale Pollendina horizontally
6. Record metrics for authorization/signing attempts
7. Blackhole clients that submit bad CSRs or tokens beyond some threshold
