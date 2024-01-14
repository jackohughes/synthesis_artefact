# Installing the artefact 

The artefact is packaged as an image of a Docker container and must be extracted
and loaded into Docker in order to run the benchmark tool. We provide steps on
how to do so in this file. 

## 1. Installation 

First verify the integrity of the archive with `md5sum` or `md5` (depending on
the user's system):

    md5sum synthesis-artefact.tar.gz
    a00191ce87b87ee0635a028dae56f776 synthesis-artefact.tar.gz

or 

    md5 synthesis-artefact.tar.gz
    MD5 (synthesis-artefact.tar.gz) = a00191ce87b87ee0635a028dae56f776

check that the hash is a00191ce87b87ee0635a028dae56f776.  

### 1.1 Using Docker commands 

Extract `synthesis-artefact.tar.gz` 
to `synthesis-artefact.tar` using the command:

    tar -xf synthesis-artefact.tar.gz    

Next, load the container image into docker with: 

    docker load < synthesis-artefact.tar
    Loaded image: synthesis-artefact:dev

That's it! The `grenchmark` can now be run from the container image. See Section
2 in the README for steps on how to do this.

### 1.2 Using the `install` script

Simply run `./install` to run the commands from the above section automatically. 

## 2. Uninstallation

After finishing with the artefact, the loaded image can be removed either by
running the docker command: 

    docker image rm synthesis-artefact:dev

or by running `./install -u`.

## 3. (Optional for M1 Mac users) Building Granule and running `grenchmark`

Unfortunately, the container runs very slowly on `aarch64` systems due to
emulation to `amd64`. If the user has no option other than using such a system,
then we suggest instead building Granule from source and running `grenchmark`
from inside the `granule` repo. We appreciate that this is not ideal and a last
resort. We provide steps on how to do so in this section should the user wish to
do so. 

First go to the [Granule repo on
github](https://github.com/granule-project/granule) and follow the instructions
there under the Installation section. 

First install Granule's dependencies
[Stack](https://docs.haskellstack.org/en/stable/README/) and
[Z3](https://github.com/Z3Prover/z3).

Then run 
    git clone -b v0.9.4.1-ESOP2024-artefact https://github.com/granule-project/granule \
    && cd granule \
    && stack setup \
    && stack install

to clone and install the tag of granule that contains the code used to build
the image for this artefact. Ensure `.local/bin` is in your path and then you
can run the benchmarking tool `grenchmark` from the top level of the `granule`
repository. `grenchmark` can be used with any of the options detailed in the
README. To generate the table with the configuration from the paper, simply run
`grenchmark` with no options.


