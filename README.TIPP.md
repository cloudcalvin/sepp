------------------------------------
Summary
------------------------------------

TIPP stands for "Taxon Identification using Phylogenetic Placement". TIPP classifies unknown fragmentary sequences into a given taxonomy using phylogenetic placement on a reference alignment and a reference tree. It requires that input fragments come from the same gene as the full length sequences in the reference alignment/tree. 

- Input: a placement tree `T` and a reference alignment `A` for a set of full-length sequences corresponding to a (marker) gene, a taxonomy `T'` that includes all species present in `A`, a mapping `M` between sequence names in `A` and species names in `T'`, and finally a set `X` of fragmentary sequences all coming from the same gene.
- Output: classification of each fragment in `X` according to the taxonomy `T'`, with a support value between 0 and 1 indicating the confidence in each classification.

Therefore, TIPP requires that the set of input fragments `X` belong to the reference (marker) gene. 

TIPP operates as follows.

1. Uses [SEPP](http://www.cs.utexas.edu/~phylo/software/sepp) to align and place fragments to the reference alignment and on the reference tree, in multiple ways (to account for uncertainty), and estimates support for each placement
2. Maps each placement into the taxonomy `T'` and calculats a total support for classification of the fragment at every node in the taxonomy. The support values can then be used to find the classification with highest support at every level. Classifications below a certain level can be ignored.

TIPP internally uses SEPP [[Mirarab et. al., PSB, 2012]](http://www.ncbi.nlm.nih.gov/pubmed/22174280) for phylogenetic placement. SEPP operates by using a divide-and-conquer strategy adopted from [SATe [Liu et. al., Science, 2009](http://www.sciencemag.org/content/324/5934/1561.abstract)] to improve the alignment produced by running HMMER (code by Sean Eddy). It then places each fragment into the user-provided tree using pplacer (code by Erick Matsen), or EPA. 

Developers: Tandy Warnow, Nam Nguyen, and Siavash Mirarab

###Publication:

Not published yet!

### Notes and Acknowledgment: 
- TIPP bundles the following three programs into its distribution:
  1. [pplacer](http://matsen.fhcrc.org/pplacer/)
  2. [hmmer](http://hmmer.janelia.org/)
  3. [EPA](http://sco.h-its.org/exelixis/software.html)
- TIPP uses the [Dendropy](http://pythonhosted.org/DendroPy/) package. 
- TIPP uses some code from [SATe](http://phylo.bio.ku.edu/software/sate/sate.html).

-------------------------------------
Installation
-------------------------------------
This section details steps for installing and running TIPP. We have run TIP on Linux and MAC, but have done the majority of testing on Linux machies. If you experience difficulty installing or running the software, please contact one of us (Tandy Warnow, Nam Nguyen, or Siavash Mirarab).

### Requirements:

Before installing the software you need to make sure the following programs are installed on your machine.

1. Python: Version > 2.6. 
2. Java: Version > 1.5

TIPP has dependency on some other python packages (e.g. dendropy), which it will automatically install. 

### Installation Steps:

TIPP is distributed as Python source code. Once you have the above required software installed, do the following. 

1. Obtain the latest TIPP distribution from the paper submission page (in future from a git repository). Uncompress the archive file, open a terminal, and cd into the `sepp-tipp` directory where you unzipped the zip file. 
2. Install: run ```sudo python setup.py install```. 
3. Configure: run ```sudo python setup.py config```. 

The last step creates a `~/.sepp/ directory` and puts a default config file under ~/.sepp/main.config. Since this is specific to a user, each user who runs TIPP needs to execute the last step. 

### Common Problems:

1. The steps described above require root access to the system. If you do not have root access, invoke the setup script as follows: ```python setup.py install --prefix=/some/path/on/your/system```, where `/some/path/on/your/system` is the path to a directory on your system to which you do have read and write access. If you use the `--prefix` option, you must ensure that the `lib/python2.x/site-packages` subdirectory (where `x` denotes the minor version number of your Python install) of the directory you specify following `--prefix=` is on Python's search path. To add a directory to Python's search path, modify your `$PYTHONPATH` environment variable.

2. TIPP relies on pplacer or EPA and HMMER for alignment and placement steps. These tools are packaged with TIPP. If for some reason the packaged version of HMMER, EPA, or pplacer do not run in your environment, you need to download and build those programs for your system (see below for links), and point TIPP to them. To point TIPP to your installation of epa, hmmer and pllacer modify ~/.sepp/main.config. 
  1. [pplacer](http://matsen.fhcrc.org/pplacer/)
  2. [hmmer](http://hmmer.janelia.org/)
  3. [EPA](http://sco.h-its.org/exelixis/software.html)


---------------------------------------------
Running TIPP
---------------------------------------------
To run TIPP, invoke the `run_tipp.py` script from the `bin` sub-directory of the location in which you installed the Python packages. To see options for running the script, use the command:

```
python <bin>/run_tipp.py -h
```

or simply:

```
run_tipp.py -h
```

The general command for running TIPP is:

```
python <bin>/run_tipp.py -t <placement_tree_file> -a <ref_alignment_file> -f <fragment_file> -r <raxml_info_file> -tx <taxonomy_file> -txm <mapping_taxonomy_to_gene_file> [-adt reference_tree_file>] [-A <alignment_set_size>] [-P <placement_set_size>] [-at <alignment_support>] [-pt placement_support]
```

To learn more about the input options refer to TIPP help (``run_tipp.py -h``). 

All the input parameters, except for fragment file, are specific to a marker gene. We have put together reference packages for 30 marker genes used in the TIPP paper, and have made those marker datasets available as part of our [submission web-site](http://www.cs.utexas.edu/~phylo/software/sepp/tipp-submission/).  Simulated fragments for the markers can also be downloaded from the submission website.

If you download our 30 marker genes, you can use those for running TIPP on fragments belonging to those markers.  For example To run TIPP-def, i.e. (95%,95%,100), on rspB marker genes for the simulated Illumina reads, you can use the following command:

```
run_tipp.py -t rpsB.refpkg/sate.taxonomy -a rpsB.refpkg/sate.fasta -r rpsB.refpkg/sate.taxonomy.RAxML_info -at 0.95 -pt 0.95 -tx rpsB.refpkg/all_taxon.taxonomy -txm rpsB.refpkg/species.mapping -adt rpsB.refpkg/sate.tree.ml -A 100 -f leaveout/rpsB.100bp.illumina.1 -o rpsB_run
```
(assuming rpsB.refpkg and the simulated fragments from our distriubtion are under the current directory)

Note that here, the placement tree, given using `-t` is different from the alignment decomposition tree, given using `-adt`. A refined taxonomy is used as the placement tree, but the actual ML tree is used for alignment decomposition. 

To run TIPP-large (described in manuscript) on 16S bacteria, you can use the following command:

```
run_tipp.py -t 16S_bacteria.refpkg/sate.taxonomy -a 16S_bacteria.refpkg/sate.fasta -r 16S_bacteria.refpkg/sate.taxonomy.RAxML_info -at 0.95  -pt 0.95 -tx  16S_bacteria.refpkg/all_taxon.taxonomy -txm 16S_bacteria.refpkg/species.mapping -A 100 -P 1000 -f leaveout/16S_bacteria.100bp.illumina.1 -o 16S_bacteria_run
```

Note that in this example, the alignment decomposition tree is not given.  This is because when placing on placement subsets, we require that the alignment decomposition tree to be identical to the placement tree.  This restriction that the alignment decomposition tre match the placement tree is purely artificial; we have not had time to implement this feature yet.  This feature is currently under development.

TIPP can also be run using a configuration file. Two sample configuration files can be found under `test/unittest/data/tipp` directory in the distribution package. To run TIPP using command options, using a configuration file similar to the command used before on rspB gene, run:

```
run_tipp.py -c test.pplacer.config
```

By default TIPP uses pplacer for the phylogenetic placement step. But it can also use EPA. Currently, the only way to instruct TIPP to use EPA is by usign a configuration file. Run the following command in order to use EPA instead of pplacer. 

```
python <bin>/run_tipp.py -c test.epa.config
```

( note that EPA is currently bundled only for Linus. On MAC, you can install your own version of EPA, and point TIPP to your version by modifying the `~/.sepp/main.config` file.) 

## TIPP Output

The main output of TIPP is a file called *<output_prefix>_classification.txt*. In this file, each line shows a potential classification of a fragment and its probability. Each line is a comma-separated list of values. The first value is the fragment name, the second value is the taxonid (according to input taxonomy) for the classification, the third value is the name of the OTU, the fourth value is the taxonomic rank, and the last value is the TIPP-estimated probability that the fragment belongs to that OTU. For example, ``EAS25_26_1_75_674_1432_0_2,28890,Euryarchaeota,phylum,.6141`` means TIPP estimates that there is a 61% chance that fragment *EAS25_26_1_75_674_1432_0_2* belong to phylum *Euryarchaeota* from the NCBI taxonomy (which has taxon id 28890). 


Since TIPP is based on running SEPP, it also produces extended alignments and placements of the fragments on the placement tree. These outputs are generated in the same directory as the main output, with names *<output_prefix>_placement.json* and *<output_prefix>_alignment.fasta*. Extended alignments are in Fasta format. Placement results are in the *jplace* format. Please refer to [pplacer website](http://matsen.github.com/pplacer/generated_rst/pplacer.html#json-format-specification '(currently here)') for more information on the format of the jplace files. Also note that pplacer package provides a program called guppy that can read .json files and perform downstream steps such as visualization.

By setting TIPP_DEBUG environmental variable to "True", you can instruct TIPP to output more information that can be helpful for debugging.  

---------------------------------------------
Bugs and Errors
---------------------------------------------
TIPP is under active research development at UTCS by the Warnow Lab (and especially with her PhD students Siavash Mirarab and Nam Nguyen). Please report any errors to Siavash Mirarab (smirarab@gmail.com) and Nam Nguyen (namphuon@cs.utexas.edu).