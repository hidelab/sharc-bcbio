# BCBIO at sharc

bcbio-nextgen, github URL, is installed on `sharc`.

It is installed in `/usr/local/community/bcbio-nextgen/`

Note that you cannot access this directory from the sharc login
node.
You must open a session on a worker node first,
for example, by running `qrshx`.

There will be several versions.

The current version is at `/usr/local/community/bcbio-nextgen/2017-08`.

You can run it by naming it directly:

    /usr/local/community/bcbio-nextgen/2017-08/tool/bin/bcbio_nextgen.py

or by adding `/usr/local/community/bcbio-nextgen/2017-08/tool/bin`
to your `PATH`, and then running `bcbio_nextgen.py`.

I've not written a module yet, but when it is written you will be
able to use the module system to add it to your `PATH` as well:

    module add blah blah blah

## A very brief intro

More comprehensive documentation is available at
 http://bcbio-nextgen.readthedocs.io/en/latest/contents/testing.html

To run the pipeline you need:
- fastq files (your bulk RNA data or whatever);
- A CSV file that names your sample files;
- a YAML template.

`bcbio_nextgen.py` is then used to convert the template and sample names into a full configuration file.
For example

    bcbio_nextgen.py -w template rnaseq-seqc.yaml input/seqc.csv input

Then you can run the analysis:

    bcbio_nextgen.py ../config/seqc.yaml

# Test materials

The `test-rnaseq` subdirectory of
 `/usr/local/community/bcbio-nextgen/2017-08` contains
 the complete results
of running the [RNAseq test example from bcbio's documentation,](http://bcbio-nextgen.readthedocs.io/en/latest/contents/testing.html#rnaseq-example).


# System configuration

Installation configuration file is at
`/usr/local/community/bcbio-nextgen/2017-08/data/config/install-params.yaml`.

Global Configuration file `bcbio_system.yaml` is at blah

Genomes: run `bcbio_nextgen.py upgrade -h` to generate the
genome list:

Aligners:

Further bcbio documentation is at http://bcbio-nextgen.readthedocs.io/en/latest/contents/configuration.html

Recommend script to invoke is:

Tests:

Install scripts:


