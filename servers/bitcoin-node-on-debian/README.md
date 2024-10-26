# Bitcoin node, running on Debian

## Introduction

The purpose and scope of this guide is to assist in the creation and configuration of a Bitcoin node.

A Bitcoin node is a server that providers to its users the ability to check transactions and review block history. While a Bitcoin node running Bitcoin Core is useful on its own, its value is magnified when using additional services and software that leverage a locally hosted timechain.

This guide has been written to simplify the installation of a Bitcoin node running Debian. In addition, this guide assumes installation on a dedicated computer; specifically, a laptop. As of the creation of this guide (2024 May 21), this entire installation requires 820GB of storage space, with Bitcoin Core requiring at minimum 50GB annually. This average only considers Bitcoin Core, not the size requirement of the transaction indexer, as is based on the assumption that the Bitcoin network appends 50,000 blocks a year, of a size of 1MB each. These days, this is a under-estimation. However, the calculation justifies the suggestion that a Bitcoin node should be created with *at minimum* 2TB of storage space.

## Table of contents

- [Introduction](#introduction)
- [Initial configuration](initial-configuration.md)
- [Security](security.md)
- [Configuring nginx](configuring-nginx.md)
- [Configuring Tor](configuring-tor.md)
- [Installing Bitcoin Core](installing-bitcoin-core.md)
- [Installing Fulcrum (Bitcoin transaction indexer)](installing-fulcrum-indexer.md)
- [Installing Mempool.space (Timechain explorer)](installing-mempool.md)
- [References](#references)

## References

- [Raspibolt documentation](https://raspibolt.org/)
    - The primary reference for many aspects of this guide.
- [YouTube - Ministry of Nodes - Ubuntu Node Box playlist](https://www.youtube.com/playlist?list=PLCRbH-IWlcW290O0N0lQV6efxuCA5Ja8c)
    - The original reference for some aspects of this guide.
