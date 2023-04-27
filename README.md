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
created by the scripts is not a secret, and can even be made publicly
available.
