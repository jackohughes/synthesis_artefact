# ESOP 2024 artefact for submission 1938

## 1. Overview
This artefact contains an image of Granule with a tool to run the suite of
synthesis benchmarks used for the evaluation in Section 5 of the paper. 

The primary purpose of this artefact is to reproduce Table 1 from Section 5
(page 24) of the paper, however, the tool can run several different benchmark
configurations and the user can customise the output of the tool. 

The benchmark tool settings that can be configured include:

- How many repeat attempts a benchmark should run for. By default this is 10.
- How long each benchmark attempt should run for before timing out. By default
  this is 10 seconds.
- Which categories of benchmarks should be run. The available categories are
  List, Stream, Bool, Maybe, Nat, Tree, and Misc. 
- Which individual benchmarks should be run, e.g. if the user only wants to
  benchmark 1 file.

By default, these values are the same as those used in the paper, i.e. 

- 10 repeat attempts per benchmark 
- 10 second timeout 
- All categories 
- All files

The tool also allows the user to supply several optional flags controlling the
output of the benchmarking tool. These include:

- The verbosity of the output. The tool can optionally print out the aggregated
  measurement results for each synthesis benchmark. 
- The tool can optionally print out the synthesised program as part of its
  output.

## 2. Getting started 

We begin by explaining how to install and run a short instance of the
benchmarking tool using Docker commands in Section 1.1. Instead of running the
full benchmarking suite, we will just run it for the List category of programs
with only 1 synthesis attempt per benchmark.

We also supply a bash script `run-benchmarks` which automates
this process, described in Section 1.2. 

### 2.1. Using Docker Commands

#### 2.1.1 Verify the image archive checksum

#### 2.1.2. Extract the image archive 
First, inside the `/artefact/` directory, extract `granule-synthesis-benchmarks.tar.gz` 
to `granule-synthesis-benchmarks.tar` using the command:

    tar -xf granule-synthesis-benchmarks.tar.gz    

#### 2.1.3. Load the image into docker
Next, load the image into docker:

    docker load < granule-synthesis-benchmarks.tar

This prints the following output to `stdout`:

    Loaded image: benchmarks-artefact:dev

where `benchmarks-artefact:dev` is the repository name and tag separated by 
a colon.

#### 2.1.4. Create the docker container 
Next, create the container passing in the repository:tag identifier
`benchmarks-artefact:dev` from above using the command:

    docker create -q -it benchmarks-artefact:dev -a 1 -c "List"

The `-a 1` flag is passed to our synthesis benchmarking tool `grenchmark` which
sets the number of synthesis attempts per benchmark to 1. The `-c "List"` flag
tells `grenchmark` to only run the `List` benchmark category.

The `docker create` command will print the container ID to `stdout`, looking
something like:

    6d8af538ec541dd581ebc2a24153a28329acb5268abe5ef868c1f1a261221752

We will henceforth refer to this as CONTAINER_ID.

#### 2.1.5. Start the docker container 
Copy the container ID returned by `docker create` and pass it into the `docker
start` command:

    docker start -i CONTAINER_ID

This will then run the `grenchmark` tool in the container with our supplied
configuration. This should take under 1 minute to run. We will explain the
output produced by `grenchmark` in depth in Section 2. For now, we verify that
it's working correctly by checking that the output looks something like:

    example of grenchmark output here

#### 2.1.6. Copy the results table PDF 

The `grenchmark` tool will generate a PDF file containing a table of benchmark
results. After `docker start` has finished, copy the resulting PDF to your
machine using:

    docker cp CONTAINER_ID:results.pdf .

This will create a file called `results.pdf` in the `/artefact/` directory. Open
the PDF and verify the table has been generated. The table will look like: 

<!-- ![Results Table Demo](/demo-table.png "table") -->

The log file produced by `grenchmark` can also be copied to the `/artefact/`
directory using: 

    docker cp CONTAINER_ID:benchmark.log .

#### 2.1.7. Remove the docker container

On successfully copying over the PDF (and/or log file), the container can be 
removed using: 

    docker rm -v CONTAINER_ID

### 2.2. Using `run-benchmarks`
The user can run this script with:

    ./run-benchmarks <OPTIONS>

The script includes a flag `-d` or `--demo` which runs a quick
configuration of the tool to check that it installs and runs correctly:

    ./run-benchmarks -d 

As in Section 2.1 this mode runs all the benchmarks for the List category of
programs only, with 1 attempt per benchmark. This is considerably faster than
running the full suite to generate the paper from the table, so it is useful to
run first.

This script will automatically load the docker image, create and start the
container, copy the results table PDF and log file to the `/artefact/`
directory, before removing the container.  

## 3. Reproducing the full results table

### 3.1. Using docker commands

To reproduce the table of results from the paper, repeat the above steps up
to and including 2.1.3. When creating the docker container, however, simply 
create it with no additional arguments:

    docker create -q -it benchmarks-artefact:dev

The default values for running the benchmarks as per the table will be used by
`grenchmark`. Then repeat steps from 2.1.5 onwards to start the container, copy
the file(s) to the host machine, and remove the container.

### 3.2. Using `run-benchmarks` 

Alternatively, simply run the bash script with no arguments:

    ./run_benchmarks

The resulting PDF will be copied over by the script with a filename
`results-YYYY-MM-DD-hh-mm.pdf`, i.e. with a datetime stamp. 

The log file `benchmarks-YYYY-MM-DD-hh-mm.log` will be copied over with the same
format. 

## 4. Understanding `grenchmark` output and table

While running, `grenchmark` will print the status of the benchmarking to
`stdout`. This section describes this output in Section 3.1 and how to read the
table of results in Section 3.2

### 4.1 The `grenchmark` output

The tool runs the benchmark tests in order of synthesis mode. As 
described in Section 5 of the paper, there are two modes of synthesis:
- Graded: This mode synthesise a program from a graded type, using grade
  information to prune the search space of programs, via SMT constraint solving.
- Cartesian: This mode synthesis a program ignoring the grades, i.e. solely from
  the type. Therefore no SMT solving is required but we discard the output if it
  does not type check and re-synthesise a program until it does.

The tool collates the benchmark tests and runs them by category first in the
Graded mode and then in the Cartesian mode. As mentioned in Section 1, the
categories of benchmark programs are `List`, `Stream`, `Bool`, `Maybe`, `Nat`,
`Tree`, and `Misc`. 

The tool then proceeds through the list of programs to benchmark, repeating each
benchmark for the configured number of repeat attempts (default 10). 

While running, the tool prints the program it is currently benchmarking, along
with status information showing how many programs it has left to benchmark, along 
with percentage progress information: 

    example of output here

For each repeat attempt, the tool indicates which attempt it is on per benchmark. 
For example if we specified 5 repeat attempts per benchmark (by passing in `-a 5`),
the output for benchmarking the `append` program in the `List` category would be: 

    example of output here

After completing each benchmark, the tool builds a LaTeX table from the results
and runs `pdflatex` to generate a PDF file of the table. The tool indicates
whether this task was successful or not.

### 4.2.1 Additional output information

As mentioned, the user can optionally supply two flags to `grenchmark`, which instruct 
it to print additional output information while carrying out the benchmarks:
- `-v, --verbose`: This flag prints the aggregated synthesis measurements for
  each benchmark program after the specified number of repeat attempts. This is
  the data that is used to construct the results table. 
  
  For example, using `-v -a 5` as arguments to `grenchmark` would print the following 
  after carrying out the benchmark for `append` in `List`: 

        example of output here

- `-s, --show-program`: This flag displays the pretty-printed Granule program that 
  has been synthesised for each program (if synthesis succeeded without timing out).

The above flags can be used in combination to achieve an output such as:

    example of output here

for the arguments `-v -s -a 10`. 

### 4.2 The results table


## 5. Other configurations 


## 6. Acknowledgements

