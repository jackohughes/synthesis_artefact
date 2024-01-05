# Requirements 

## OS 

We highly recommend that the user runs this artefact on an `amd64` architecture.
Otherwise the emulation into `aarch64` (e.g. for an M1 Mac) will make the
benchmarking suite run much slower. 

If the user does not have access to an `amd64` machine, we would suggest
building `granule` from source and running the benchmarking tool `grenchmark`
directly (although we appreciate this is not ideal). We provide steps on how to
do this in Section 3 of the INSTALL file, should the user wish to do so.

## Software

The main software requirement to use this artefact is Docker. The artefact is
installed and run via Docker CLI commands. 

Alternatively Podman could be used with the equivalent commands (although the
`install` and `run` scripts only works with Docker).

A PDF viewer is also required to view the table of results produced by the
artefact.

## Storage

The container image is ~4GB once extracted so we recommend having at least 4.5GB
available storage capacity.


