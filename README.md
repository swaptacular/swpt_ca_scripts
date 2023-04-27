# Scripts for managing Swaptacular certificate authorities

This repository contains shell scripts that Swaptacular network nodes can
use to maintain a trusted certificate authority (*root-CA*). The way this
works [is explained
here](http://swaptacular.github.io/2023/04/26/under-the-hood-peer-connections/).
The scripts should work on any contemporary GNU or BSD system, on which
`bash` is installed, along with `openssl`, `zip`, `unzip`, `envsubst`, and
`hexdump`.

## Creating the root-CA database

To create your root-CA, first you need to generate a private key for it. To
do this, use the `generate-masterkey` script:

```shell
$ ./generate-masterkey
```

This will create a `private/` sub-directory which will contain your root-CA
private key, encrypted with a password that you will be asked to enter.
**You should keep the content of the `private/` sub-directory secret**. The
content of any other files and sub-directories, that are automatically
created by the scripts, is not a secret, and can even be made publicly
available.

Once you have successfully generated a private key, use the `init-ca`
command to create the root-CA database:

```shell
$ ./init-ca
```

You will be asked a bunch of questions, and you will have to enter the
password for your private key a few times. At the end, a self-signed
certificate will be generated for your certificate authority. Also, a bunch
of sub-directories will be created. For example, after running the `init-ca`
command, your script directory will contain something similar to this:

``` shell
$ ls -F
certs/               generate-serverkey*  peers/          root-ca.conf.template
create-infobundle*   init-ca*             private/        root-ca.crt
db/                  LICENSE              README.md       sign-peercert*
generate-masterkey*  nodeinfo/            register-peer*  sign-servercert*
```

* `root-ca.conf.template` contains configuration parameters for the OpenSSL
  library. This file is used only during the initial creation of the root-CA
  database.
* `root-ca.crt` contains the self-signed certificate for your root-CA.
* `certs/` will contain the certificates that have been signed by your
  root-CA.
* `db/` will contain all sorts of bookkeeping information about your
  root-CA.
* `nodeinfo/` will contain information about your Swaptacular node. You can
  add random files to this directory, and your peers will store all those
  files in their root-CA databases. Most importantly, the information that
  you put here, should allow your peers to get in touch with you, if
  necessary.
* `peers/` will contain information about your peers, including the content
  of their `nodeinfo` directories.

As you may have guessed, the information contained in these files and
directories **is very important** for the proper functioning of your
Swaptacular node. Therefore, it is probably a good idea to use a version
control system (like `git`), and each time you add a new peer, or make other
important changes, to commit those changes to your version control servers.

## Creating an "info-bundle" file

Every Swaptacular network node should create an "info-bundle" file for
itself, which is a `.zip` file that contains 4 files:

1. The self-signed certificate for the node's root-CA (`root-ca.crt`).
2. A certificate signing request (`root-ca.csr`).
3. The contents of the `nodeinfo` sub-directory (`nodeinfo.zip`).
4. A digital signature for the `nodeinfo.zip` file (`nodeinfo.signature`).

To create your "info-bundle" file, use the `create-infobundle` command,
specifying the name of the info-bundle file that should be created
("my-foo-node", in this example):

```shell
$ ./create-infobundle my-foo-node
```
