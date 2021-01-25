
## bcov

A tool for efficient binary-level coverage analysis. **bcov** statically
instruments x86-64 ELF binaries without compiler support. It features probe
pruning, precise CFG analyses, and sophisticated instrumentation techniques.
We summarized this research in a 2-min teaser [video][link-teaser].

## Resources

  - Details are available in our ESEC/FSE'20 [paper][link-paper]. You can find a slightly expanded pre-print [here][link-preprint].
  - We have a [post][link-llvmdev] on the llvm-dev mailing list. It briefly introduces **bcov** and discusses potential future work.
  - This blog [post][link-post1] elaborates on the availability of function definitions in stripped binaries.
  - You can find our ESEC/FSE'20 talk [here][link-talk]. The slide deck (with transcript) is also [available][link-slide-deck].
  - A sample set of binaries that we patched using `bcov` is available [here][link-samples]. However, the complete set of benchmarks is available on our archival [repository][link-benchmarks].


## Software prerequisites

The following software packages must be available:
   - [capstone][link-capstone] branch `next` commit #c3b4ce1901
   - [unicorn][link-unicorn]  branch `master` commit #536c4e77c4

Later versions of both frameworks should work in principle but have not been
tested yet. The script `install.sh` can be used for installation.

## Research reproducibility

We provide a [Dockerfile][link-dockerfile] which installs **bcov** and runs a coverage analysis experiment.
Please checkout the supplemental [artifacts][link-artifacts] for more details.

## Usage

The tool supports the following operation modes which are set using the option `--mode` (or simply `-m`):

  - `patch`. Patch a given binary.
  - `report`. Report coverage given a patched binary and a coverage data file.
  - `dump`. Dump various program graphs for a given function. For example, dump the CFG and dominator trees.

The following command can be issued to patch a binary,

```shell script
bcov -m patch -p any -v 5 -i perl -o perl.any
```

The instrumentation policy can be set to `any`, which refers to the any-node policy, or `all` which
refers to the leaf-node policy.

Coverage data can be dumped by injecting `libbcov-rt.so` using the `LD_PRELOAD`
mechanism. For example, you can try the sample binary `perl.any` which can be
found in the artifacts repository,

```shell script
export BCOV_OPTIONS="coverage_dir=$PWD"   # sets the directory for dumping coverage data. Defaults to $PWD
export LD_PRELOAD="[full-path-to-bcov-rt]/libbcov-rt.so"
./perl.any -e 'print "Hello, bcov!\n"'
```

This will produce a dump file that has the extension '.bcov' in your current
directory. This file can be supplied to **bcov** for coverage reporting,

```shell script
bcov -m report -p any -i ./perl -d perl.any.1588260679.1816.bcov > report.out
```

Currently, **bcov** can not persist analysis results to disk. Therefore, the original binary must be
re-analyzed to report coverage. Coverage will be reported for each basic block in the file
`report.out`. The data in each line lists:
 - BB address
 - BB instruction count
 - is covered
 - is fallthrough (i.e., does not terminate with a branch)

Also, a coverage summary is reported for each function. For example, it shows the basic block and instruction coverage ratios.

For a given function, it is possible to selectively dump various program graphs like the CFG and superblock dominator graph.
For example, consider function `S_search_const` in `perl`,

```shell script
bcov -m dump -f "S_search_const" -i ./perl
```

This command will dump the following files:

   - func_421d90.cfg.dot. The CFG of the function.
   - func_421d90.rev.cfg.dot. Similar to the CFG but with all edges reversed.
   - func_421d90.pre.dom.dot. Predominator tree.
   - func_421d90.post.dom.dot. Postdominator tree.
   - func_421d90.sb.dom.dot. Superblock dominator graph.

Graphs are dumped in the standard DOT format and can be viewed using a dot viewer like `xdot`.
Please refer to this [blog post][link-post2] for additional details.

## Citing

For citation in an academic work please use:

    @inproceedings{BenKhadra:FSE2020,
    address = {Virtual Event, USA},
    author = {{Ben Khadra}, M. Ammar and Stoffel, Dominik and Kunz, Wolfgang},
    booktitle = {ACM Joint European Software Engineering Conference and Symposium on the Foundations of Software Engineering - ESEC/FSE'20},
    doi = {10.1145/3368089.3409694},
    pages = {1153--1164},
    publisher = {ACM Press},
    title = {{Efficient Binary-Level Coverage Analysis}},
    year = {2020},
    month = {nov},
    day = {6--13}    
    }

## License

This software is distributed under the MIT license. See `LICENSE.txt` for details.

 [link-capstone]: https://github.com/aquynh/capstone
 [link-unicorn]: https://github.com/unicorn-engine/unicorn
 [link-preprint]: https://arxiv.org/pdf/2004.14191.pdf
 [link-artifacts]: https://github.com/abenkhadra/bcov-artifacts
 [link-post1]: https://blog.formallyapplied.com/2020/05/function-identification/
 [link-post2]: https://blog.formallyapplied.com/2020/06/bcov-program-graphs/
 [link-dockerfile]: https://github.com/abenkhadra/bcov-artifacts/blob/master/Dockerfile
 [link-llvmdev]: http://lists.llvm.org/pipermail/llvm-dev/2020-June/142821.html
 [link-slide-deck]: https://docs.google.com/presentation/d/1BgcOaw06AcfCzc8PMPkhYANKUkevbxAPFQkbERpXBJ4/edit?usp=sharing
 [link-talk]: https://youtu.be/pns0lJ5FwwA
 [link-teaser]: https://youtu.be/AT29eitsChc
 [link-paper]:  https://doi.org/10.1145/3368089.3409694
 [link-samples]: https://github.com/abenkhadra/bcov-artifacts/tree/master/sample-binaries
 [link-benchmarks]: https://doi.org/10.5281/zenodo.3876047
