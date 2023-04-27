# Scripts for managing Swaptacular certificate authorities

This repository contains shell scripts that Swaptacular network nodes can
use to maintain a trusted certificate authority (*root-CA*). The way this
works is explained
[here](http://swaptacular.github.io/2023/04/26/under-the-hood-peer-connections/).
The scripts should work on any contemporary GNU or BSD system, on which
`bash` is installed, along with `openssl`, `zip`, `unzip`, `envsubst`, and
`hexdump`.

To create your root-CA, first you need to generate a private key for it. To
do this, use the `generate-masterkey` script:

```shell
$ ./generate-masterkey
```
