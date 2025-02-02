# Nextstrain + Phytest

[![CI](https://github.com/phytest-devs/phytest-nextstrain-example/actions/workflows/CI.yaml/badge.svg)](https://phytest-devs.github.io/phytest-nextstrain-example/)

This repository provides an example of using [phytest](https://github.com/phytest-devs/phytest) for quality control in a Snakemake pipeline. Phytest allows us to write tests for our pipeline the same way we write test for our code. This repo builds on the data and scripts associated with the [Zika virus tutorial](https://nextstrain.org/docs/getting-started/zika-tutorial). We add phytest to the pipeline to ensure our alignment and maximum likelihood tree meet our explicit requirements before proceeding though the pipeline.

![](dag.svg)

For more information see the [phytest](https://github.com/phytest-devs/phytest) docs.

## Phytesting

We have added the following rule to the Snakemake workflow. We use phytest to run our tests on the alignment and the tree, outputting a HTML report. Only if all our tests past will we continue through the pipeline. 

```yml
rule phytest:
    message: "Running phytest"
    input:
        alignment = rules.align.output.alignment,
        tree = rules.tree.output.tree,
        phytest = phytest_file
    output:
        report = report("results/phytest-report.html")
    shell:
        """
        phytest {input.phytest} -s {input.alignment} -t {input.tree} --report {output.report}
        """
```

In the `config/test.py` file we define our tests that the alignment and tree must pass. 

```python
from phytest import Alignment, Sequence, Tree

def test_alignment_size(alignment: Alignment):
    """
    By testing the size of our alignment we can 
    ensure that our pipeline won't run if there are 
    accidental sequence additions or deletions.
    """
    alignment.assert_length(34)
    alignment.assert_width(10769)

def test_sequence_no_gaps(sequence: Sequence):
    """
    We know that our sequences shouldn't have any
    gaps so we enforce this constrain to ensure the 
    quality of our alignment.
    """
    sequence.assert_count_gaps(0)

def test_tree_is_SG_monophyletic(tree: Tree):
    """
    We expect that all the samples from Singapore will 
    form a monophyletic clade in the tree.
    """
    singapore_tips = [tip for tip in tree.get_terminals() if 'SG_' in tip.name]
    tree.assert_is_monophyletic(singapore_tips)

```


## Usage 

Create a conda environment named `nextstrain` and install all the necessary software using conda (or mamba):

```bash
conda env create -n nextstrain --file env.yaml
```

Activate the conda environment:

```bash
conda activate nextstrain
```

Run the pipeline 

```
snakemake -c 1 
```

To generate an HTML report with the Phytest report embedded in it, run this command after the workflow has finished:

```
snakemake -c 1 --report report.html
```

Then open the `report.html` in a browser. An example of a this report is generated as part of the [CI for this repository](https://phytest-devs.github.io/phytest-nextstrain-example/).
