# ESOP 2024 artefact for submission 1938

## 1. Overview
This artefact contains an image of Granule which runs the suite of benchmark 
tests used for the evaluation in Section 5 of the paper. 

The primary purpose of this artefact is to reproduce Table 1 from Section 5 (page 24), 
however, the tool can run several different benchmark configurations. 

The tool is configurable, allowing the user to specify several properties of 
the benchmarking, including:
- How many repeat attempts a benchmark should run for. By default this is 10.
- How long each benchmark attempt should run for before timing out. By default this is 10 seconds.
- Which categories of benchmarks should be run. The available categories are List, Stream, Bool, Maybe, Nat, Tree, and Misc. 
- Which individual benchmarks should be run, e.g. if the user only wants to benchmark 1 file.

By default, these values are the same as those used in the paper, i.e. 
- 10 repeat attempts per benchmark 
- 10 second timeout 
- All categories 
- All files

The tool also allows the user to supply several optional flags controlling the output 
of the benchmarking tool. These include:

- The verbosity of the output

## 2. Getting started 

We begin by explaining how to install and run a short instance of the
benchmarking tool using Docker commands in Section 1.1. Instead of running the
full benchmarking suite, we will just run it for the List category of programs
with only 1 synthesis attempt per benchmark.

We also supply a bash script `run-benchmarks` which automates
this process, described in Section 1.2. 

### 2.1. Using Docker Commands

#### 2.1.1. Extract the image archive 
First, inside the `/artefact/` directory, extract `granule-synthesis-benchmarks.tar.gz` 
to `granule-synthesis-benchmarks.tar` using the command:

    tar -xf granule-synthesis-benchmarks.tar.gz    

#### 2.1.2. Load the image into docker
Next, load the image into docker:

    docker load < granule-synthesis-benchmarks.tar

This prints the following output to `stdout`:

    Loaded image: benchmarks-artefact:dev

where `benchmarks-artefact:dev` is the repository name and tag separated by 
a colon.

#### 2.1.2. Create the docker container 
Next, create the container passing in the repository:tag identifier
`benchmarks-artefact:dev` from above using the command:

    docker create -q -it benchmarks-artefact:dev -a 1 -c "List"

The `-a 1` flag is passed to `grenchmark` which sets the number of synthesis
attempts per benchmark to 1. The `-c "List"` flag tells `grenchmark` to only run
the `List` benchmark category.

The `docker create` command will print the container ID to `stdout`, looking
something like:

    6d8af538ec541dd581ebc2a24153a28329acb5268abe5ef868c1f1a261221752

We will henceforth refer to this as CONTAINER_ID.

#### 2.1.2. Start the docker container 
Copy the container ID returned by `docker create` and pass it into the `docker
start` command:

    docker start -i CONTAINER_ID

This will then run the `grenchmark` tool in the container with our supplied
configuration. This should take under 1 minute to run. We will explain the
output produced by `grenchmark` in depth in Section 2. For now, we verify that
it's working correctly by checking that the output looks something like:

    example of grenchmark output here

#### 2.1.3. Copy the results table PDF 

The `grenchmark` tool will generate a PDF file containing a table of benchmark
results. After `docker start` has finished, copy the resulting PDF to your
machine using:

    docker cp CONTAINER_ID:results.pdf .

This will create a file called `results.pdf` in the `/artefact/` directory. Open
the PDF and verify the table has been generated. The table will look like: 

<!-- ![Results Table Demo](/demo-table.png "table") -->

The log file produced by `grenchmark` can also be copied to the `/artefact/` directory 
using: 

    docker cp CONTAINER_ID:benchmark.log .

#### 2.1.4. Remove the docker container

On successfully copying over the PDF (and/or log file), the container can be 
removed using: 

    docker rm -v CONTAINER_ID

### 2.2. Using `run-benchmarks`
The user can run this script with:

    ./run-benchmarks <OPTIONS>

The script includes a flag `-d` or `--demo` which runs a quick
configuration of the tool to check that it installs and runs correctly:

    ./run-benchmarks -d 

This "mode" runs all the benchmarks for the List category of programs only, with
1 attempt per benchmark. This is considerably faster than running the full suite
to generate the paper from the table, so it is useful to run first.

## 3. Understanding `grenchmark` output

## 4. Reproducing the full results table

### 4.1. Using docker commands

To reproduce the table of results from the paper, simply run the artefact with no 
arguments:

The default values for running the benchmarks as per the table will be loaded by
`grenchmark`. After the container finishes running, copy the generated PDF file
from the container using the `docker cp` command:

before removing the container:

The resulting PDF file will be available at `results.pdf` in the `/artefact/` 
directory. The log file produced by `grenchmark` can also be copied over with: 

### 4.2. Using `run-benchmarks` 

Alternatively, simply run the bash script with no arguments:

    ./run_benchmarks

The resulting PDF will be copied over by the script with a filename
`results-YYYY-MM-DD-hh-mm.pdf`, i.e. with a datetime stamp. The script will
automatically remove the container. 

A log file `benchmarks-YYYY-MM-DD-hh-mm.log` will be copied over with the same
format. 

## 5. Other configurations 


## 6. Acknowledgements

