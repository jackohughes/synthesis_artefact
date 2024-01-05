# ESOP 2024 artefact for submission 1938

## 1. Overview
This artefact contains an image of a container with a tool to run the suite of
synthesis benchmarks in Granule used for the evaluation in Section 5 of the
paper. 

The primary purpose of this artefact is to reproduce Table 1 from Section 5
(page 24) of the paper, however, the tool can run several different benchmark
configurations and the user can customise the output of the tool. The benchmark
tool settings that can be configured include:

- How many repeat attempts a benchmark should run for. By default this is 10.
- How long each benchmark attempt should run for before timing out. By default
  this is 10 seconds.
- Which categories of benchmarks should be run. The available categories are
  `List`, `Stream`, `Bool`, `Maybe`, `Nat`, `Tree`, and `Misc`. 
- Which individual benchmarks should be run, e.g. if the user only wants to
  benchmark 1 file.

By default, these values are the same as those used in the paper, i.e. 

- 10 repeat attempts per benchmark 
- 10 second timeout
- All categories 
- All files

## 2. Getting started 

We begin by explaining how to run a short instance of the benchmarking tool
using Docker commands. Instead of running the full benchmark suite, we will just
run it for the List category of programs with only 1 synthesis attempt per
benchmark. 

To install the artefact see the INSTALL file. This is a short process 
which comprises loading the image of the container into Docker.

After installation, the tool can be run with 

    docker run --cidfile .cid -ti synthesis-artefact:dev --attempts 1 --categories "List"

which creates a container from the image and executes it with our above
mentioned configuration: `-attemtps 1` specifies only 1 attempt per benchmark
while `--categories "List"` specifies only benchmark the `List` cateogry of
programs.

This should take under 1 minute to run. We will explain the output produced by
benchmarking tool in depth in Section 3. For now, we verify that it's working
correctly by checking that the output looks like:

    Running benchmarks...
    ---------------------------
    Running benchmarks for mode: Graded
        Running benchmark 1/38 (2.63%): append
            Repeat no: [1/1]
        Running benchmark 2/38 (5.26%): concat
            Repeat no: [1/1]
    ... 

The benchmarking tool will generate a PDF file containing the table of benchmark
results. After `docker run` has finished we can copy this to the host machine.
First create a timestamp for our file using:

    TIME=$(date +"%T")

Next get the container ID by running 

    cat .cid

copy the resulting hash from `stdout` and the substitute it for CONTAINER_ID in
the command:

    docker cp CONTAINER_ID:results.pdf ./results-$TIME.pdf

This will create a timestamped PDF file in the current directory. Open the PDF
and verify the table has been generated. The table will look similar to that of
the paper but with only the `List` of programs (indicated by the vertical text
on the leftmost column). The standard sample errors (the values in brackets)
will all be zero as well as we only ran each benchmark once so no averaging took
place.

The log file produced by `grenchmark` can also be copied to the host machine by: 

    docker cp CONTAINER_ID:benchmarks.log ./benchmarks-$TIME.log

On successfully copying over the PDF (and/or log file), the container can be
removed using: 

    docker rm -v CONTAINER_ID

### 2.1. Using the `run` script

The steps above can be run automatically using the `run` script. 

    ./run [OPTIONS]

The script includes a flag `-d` or `--demo` which runs the configuration of the
benchmarking tool we used above:

    ./run --demo

This script will automatically  create and run the container, copy the results
table PDF and log file to the current directory, before removing the container.  

## 3. Reproducing the full results table

To reproduce the table of results from the paper simply run either 

    docker run --cidfile .cid -ti artefact:dev

or 

    ./run

The default values for running the benchmarks as per the table will be used by
benchmarking tool. If the first of these options was done, repeat steps from the
previous section to copy the PDF (and log) over to the host machine and remove
the container.

## 4. Understanding the output and table

While running, the benchmarking tool will log the status of the benchmarking to
`stdout` and will produce a results table PDF upon conclusion. The tool's
logging output in Section 3.1 and the table of results in Section 3.2

### 4.1 The logging output

The tool runs the benchmark tests in order of synthesis mode. As 
described in Section 5 of the paper, there are two modes of synthesis:
- Graded: This mode synthesise a program from a graded type, using grade
  information to prune the search space of programs, via SMT constraint solving.
- Cartesian: This mode synthesis a program ignoring the grades, i.e. solely from
  the type. Therefore no SMT solving is required but we discard the output if it
  does not type check and keep re-synthesising a program until one type checks.

The tool collates the benchmark tests and runs them by category first in the
Graded mode and then in the Cartesian mode. As mentioned in Section 1, the
categories of benchmark programs are `List`, `Stream`, `Bool`, `Maybe`, `Nat`,
`Tree`, and `Misc`. 

The tool then proceeds through the list of programs to benchmark, repeating each
benchmark for the configured number of repeat attempts (default 10). 

While running, the tool prints the program it is currently benchmarking, along
with status information showing how many programs it has left to benchmark, along 
with percentage progress information: 

    Running benchmark 3/38 (7.89%): empty

For each repeat attempt, the tool indicates which attempt it is on per benchmark. 
For example if we specified 5 repeat attempts per benchmark (by passing in `-a 3`),
the logging output for benchmarking the `append` program in the `List` category would show: 

    Running benchmarks...
    ---------------------------
    Running benchmarks for mode: Graded
        Running benchmark 1/2 (50.00%): append
            Repeat no: [1/3]
            Repeat no: [2/3]
            Repeat no: [3/3]
    ---------------------------
    Running benchmarks for mode: Cartesian
        Running benchmark 2/2 (100.00%): append
            Repeat no: [1/3]
            Repeat no: [2/3]
            Repeat no: [3/3]
    ---------------------------

After completing each benchmark, the tool builds a LaTeX table from the results
and runs `pdflatex` to generate a PDF file of the table. The tool indicates
whether this task was successful or not:

    Benchmarks complete. Creating table of results...
    LaTeX table created.

    Generating PDF...
    Done: PDF generated at results.pdf

### 4.1.1 Additional output information

As mentioned, the user can optionally supply two flags to the tool, which
instruct it to print additional output information while carrying out the
benchmarks:
- `-v, --verbose`: This flag prints the aggregated synthesis measurements for
  each benchmark program after the specified number of repeat attempts. This is
  the data that is used to construct the results table. 
  
For example, using `-v -a 1` as arguments to tool would log the
following after carrying out the benchmark for `append` in `List`, for the 
Graded mode:

    Aggregate measurements for 1 runs:
      - Context size: 0
      - Synthesis time: 181.21 (  0.00)
      - Paths explored: 130
      SMT data for graded mode:
      - Solver time: 173.16 (  0.00)
      - SMT calls made:  12
      - Mean theorem size:  31.67 (  0.00)

And for the Cartesian mode: 

    Aggregate measurements for 1 runs:
      - Context size: 0
      - Synthesis time:   6.15 (  0.00)
      - Paths explored: 130


- `-s, --show-program`: This flag displays the pretty-printed Granule program
  that has been synthesised for each program (if synthesis succeeded without
  timing out).

For example, using the options `-s -f "map:List" -a 1` will log the following output: 
    Running benchmark 1/2 (50.00%): map
        Repeat no: [1/1]
    Synthesised program for map:
        \x -> \y ->
            (case y of
                Nil -> Nil;
                Cons z u -> (Cons (x z)) ((map x) u))

### 4.2 The results table

The table in the PDF file records the results comparing grade-and-type-directed
synthesis (the Graded column) vs. purely type-directed synthesis (the Cartesian
column) where a program that doesn't type check against the original graded type
is discarded and we synthesise another program. The description from Section 5.2
provides a detailed overview of how to read the results table. We restate the
main points in this section and show how the table resulting from running the
benchmarking tool as per Section 3 of this document can be compared to the paper
table.

The left column gives the benchmark name, the number of in-scope top-level
definitions that can be used as components in the synthesis (labeled Ctxt) and
the minimum number of examples needed to synthesise the programs in the Graded
and Cartesian columns. The number in parentheses notes the number of
extra-examples required to synthesise a Cartesian program if we forgo
type-checking.

Each subsequent results column records whether the program was synthesised
succesfully or not with a tick or cross, the mean synthesis time (μT) or if a
timeout occurred and the number of branching paths explored in the synthesis
search space (Paths).

The first results column (Graded) contains the results for graded synthesis. The
second results column (Cartesian + Graded type-check) contains the re- sults for
synthesising a program in the Cartesian (de-graded) setting, using the same
examples set as the Graded column, and recording the number of retries
(consultations of the type-checker) N needed to reach a well-resourced program.
In all cases, this resulting program in the Cartesian column was equivalent to
that generated by the graded synthesis, none of which needed any retries (i.e.,
implicitly N = 0 for graded synthesis).

For each row, we highlight the column which synthesised a result the fastest in
blue and we highlight the column with the smallest number of synthesis paths
explored in yellow.

When comparing the table outputted from running the tool according to Section 4,
the main comparison to make between the table from the paper is that the
highlighted results should be equal (except perhaps in the case of benchmarks
which are very close in synthesis time across columns). 

Variations in the the synthesis time (the μT columns) between the generated
results table and the paper table are to be expected, however, the overall
trends should be the same (as indicated by the placement of the blue
highlights).

The values and highlighted results in the Paths columns should be identical. 


## 5. Other configurations 

The user can control the configuration of benchmarking tool using several flags
which we describe in this section. The flags can either be provided to the
`docker run` command or to the `run` script.

The following flags may be used to configure the tool to run other
combinations of benchmarks:
- `-c, --categories "cat_1 ... cat_n"`: This flag takes an argument which is a
  non-empty list of whitespace separated category names in quotation marks. The
  categories can be any combination of `List`, `Stream`, `Bool`, `Maybe`, `Nat`,
  `Tree`, and `Misc`. The tool will then only include these categories of
  benchmark programs when benchmarking.
- `-f, --files "name:cat_1 ... name:cat_n"`: This flag takes an argument which
  is a non-empty list of whitespace separated program/category pairs (separated
  by a colon). The program's category must be included as some benchmark
  programs have the same name in different categories, e.g. a `map` benchmark
  program appears in `List`, `Stream`, `Maybe`, and `Tree` categories. The full
  list of available benchmark program names and their categories can be seen in
  the table in the paper.

For example, if we wanted to run each program in the `Maybe` and `Misc`
categories of benchmarks along with the `append` and `map` programs from the `List`
category, we can supply the arguments to the `docker run` command: 

    docker run --cidfile .cid -ti artefact:dev -c "Maybe Misc" -f "append:List map:List"

or to the `run` script:

    ./run -c "Maybe Misc" -f "append:List map:List" 

The following options can also be used to configure the benchmarking tool:
- `-a, --attempts int`: As mentioned, this option is used to specify how many
  times we should repeat each benchmark program. Default is 10.
- `-t, --timeout int`: Used to give a custom timeout length to the tool in
  seconds. The benchmarking tool then runs each benchmark program for this
  duration, marking the benchmark program as having timed out if it exceeds this
  limit. Default is 10. 