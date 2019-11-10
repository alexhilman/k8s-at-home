# K8S at Home

K8S at Home name is documentation for myself as well as a guide to other developers they may set up their own Kubernetes infrastructure for their own use.

Kubernetes has a fairly steep learning curve; this repository contains condensed information to get bring devs up to speed farily quickly.

## Prerequisites

Before you begin, ensure you have met the following requirements:
* You have a working understanding of Linux operating systems and utilities like SSH
* You have a basic understanding of containerization; ideally with [Docker](https://docs.docker.com/)
* You have a bare-metal or vm-based Linux distribution with an [`iptables` version >= 1.6.2](documentation/iptables-race-condition.md)
    * This guide uses Ubuntu 19.10 as it has `iptables` 1.8.3

## License

This project uses the following license: [Apache 2.0 License](LICENSE)