# Scripts for managing Swaptacular certificate authorities

This repository contains shell scripts that Swaptacular network nodes can
use to maintain a trusted certificate authority (root-CA). The way this
works is explained
[here](http://swaptacular.github.io/2023/04/26/under-the-hood-peer-connections/).
The script should work on contemporary GNU or BSD systems, on which `bash`
is installed, along with `openssl`, `zip`, `unzip`, `envsubst`, and
`hexdump`.
