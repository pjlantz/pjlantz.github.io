# CVE-2021-44733: Fuzzing and exploitation of a use-after-free in the Linux kernel TEE subsystem

Recently a use-after-free vulnerability was discovered in the Linux kernel TEE subsystem, up to and including version 5.15.11, and was assigned CVE-2021-44733 (1).

At a first glance it did not seem to be exploitable for several reasons, however after some further analysis of the vulnerable code path and by implementing a crude proof-of-concept exploit it was possible to overwrite a function pointer in the kernel. No privilege escalation payload is presented in this post, however the entire environment for running OPTEE and the exploit is available for further testing, see 'Setting up the environment'.

## Background

A TEE (Trusted Execution Environment) is a trusted OS running in some secure environment, for example, TrustZone on ARM CPUs. A TEE driver handles the details needed to communicate with the TEE. Some of the more important duties of the driver is to provide a generic API towards the TEE based on the Globalplatform TEE Client API specification (2), but also to manage the shared memory between Linux and the TEE. This subsystem can be enabled by configuring CONFIG_OPTEE in the kernel configurations for ARM architectures.

The secure world contains the trusted OS denoted OP-TEE OS (4). On top of this OS it is possible to have so called Trusted Applications (TAs) running which can perform some operations in the isolated environment, see Figure 1. The normal world (Linux userspace/kernel) can interact with these applications using client applications (CAs) and the API exposed by the TEE subsystem. A CA can open a session towards a specific TA and invoke functions that the TA implements. Passing of any arguments back and forth between the TA and CA is done using shared memory.

![Session between CA and TA](/pjlantz.github.io/assets/overview.png)
