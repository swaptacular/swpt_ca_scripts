# Scripts for managing Swaptacular certificate authorities

This repository contains shell scripts that Swaptacular network nodes can
use to maintain a trusted certificate authority (*root-CA*). The way this
works [is explained
here](http://swaptacular.github.io/2023/04/26/under-the-hood-peer-connections/).
The scripts should work on any contemporary GNU or BSD system, on which
`bash` is installed, along with `openssl`, `zip`, `unzip`, `envsubst`, and
`hexdump`.

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
command to initialize the root-CA database:

```shell
$ ./init-ca
```

You will be asked a bunch of questions, and you will have to enter the
password for your private key a few times. At the end, a self-signed
certificate will be generated for your certificate authority. Also, a bunch
of sub-directories will be created, which contain the root-CA database. For
example, after running the `init-ca` command, the content of your script
directory will be:

``` shell
$ ls -F
certs/               generate-serverkey*  peers/          root-ca.conf.template
create-infobundle*   init-ca*             private/        root-ca.crt
db/                  LICENSE              README.md       sign-peercert*
generate-masterkey*  nodeinfo/            register-peer*  sign-servercert*
```

- The `certs/` sub-directory will contain the certificates that have been
  signed by your root-CA.

- The `db/` sub-directory will contain all sorts of bookkeeping information
  about your root-CA.

- The `nodeinfo/` sub-directory will contain information about your
  Swaptacular node. You can add random files to this directory, and your
  peers will store all those files in their root-CA databases. Most
  importantly, the information that you put here should allow your peers to
  get in touch with you, if necessary.

- The `peers/` sub-directory will contain information about your peers,
  including the content of their `nodeinfo/` directories.
