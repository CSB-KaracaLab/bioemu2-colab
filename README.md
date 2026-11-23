# BioEmu2 on Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/CSB-KaracaLab/bioemu2-colab/blob/main/BioEmu2_Colab.ipynb)
[![arXiv](https://img.shields.io/badge/arXiv-2611.27924-b31b1b.svg)](https://arxiv.org/abs/2611.27924)

A single notebook that runs [BioEmu2](https://github.com/microsoft/bioemu2), Microsoft Research's biomolecular emulator ([Gastegger et al., 2026](https://arxiv.org/abs/2611.27924)), on a Google Colab GPU. The code follows the usage examples in the BioEmu2 repository; this notebook turns them into a Colab form with input fields, plots and a download step, so that the model can be tried without a local GPU or a local installation.

## What BioEmu2 is

BioEmu2 ([Gastegger et al., 2026](https://arxiv.org/abs/2611.27924)) is one generative model with three modes, trained on experimental and synthetic structures and on a large body of molecular dynamics. It takes systems of proteins, nucleic acids, modified residues, ligands and ions. What the paper reports for each mode, and where this notebook runs it:

| Mode | What the paper shows | Notebook |
|---|---|---|
| **Structure prediction**: one sharp, most likely structure | Accuracy comparable to open-weight co-folding models on FoldBench and Runs N' Poses, strongest for protein–protein interfaces | step 5 |
| **Equilibrium sampling**: independent conformations of the solvated system | About half of the states MD finds for proteins and protein complexes, with 0.7–1.2 kcal/mol free-energy error on the states it does sample | step 6 |
| **Time propagation**: endpoints at a chosen lag, chained into a coarse trajectory | Matches MD autocorrelations at lags of 1 ns to 1 µs, with no clear build-up of error over many steps; binding of PUMA to MCL-1 and domain closure of adenylate kinase as examples | step 7 |

The paper also describes tools this notebook does **not** use, all available in BioEmu2 itself:

- **Contact conditioning**: residue pairs given as input, as a soft constraint. One inter-chain contact rescues a failed 4,020-residue complex prediction (8WY8, 71.2 Å → 5.0 Å RMSD).
- **Steering and enhanced sampling**: bias potentials applied while sampling, for physical geometry, chosen conformations, or rare states and free energies. The notebook's physicality step is a check after sampling, not steering.
- **Context parallelism**: one system split over several GPUs, up to about 16,000 residues. Colab gives one GPU, so the notebook stops well short of that.

On top of BioEmu2's own output, the notebook adds plots of Cα RMSD and radius of gyration over the ensemble, 3D views with py3Dmol, a zip of everything, and step 8, a check of whether a protein–protein interface is *consistent* across the ensemble, set beside MD of the same complex.

## How to use it

Open the notebook in Colab with the badge above, set **Runtime → Change runtime type → GPU**, and run the cells from top to bottom. A free T4 works for small proteins; an L4 or A100 is better for complexes. The install takes about three minutes and the weights are downloaded from Hugging Face on first use.

The system is defined either by a **PDB ID** or by **sequences** separated by `:`, with optional ligands by CCD code or SMILES. By default the multiple sequence alignment is built on the public ColabFold MMseqs2 server, which means the sequence you enter is sent to that service; you can upload your own A3M file instead.

## What to keep in mind

BioEmu2 is research software. Its model card says it is released for research purposes, not for commercial or real-world use without further testing. From the paper and the model card:

- **Frequencies are not calibrated probabilities.** The paper's free-energy errors near 1 kcal/mol are for the states BioEmu2 samples, and it samples about half of those MD visits. The model card still warns against reading sampling frequencies as Boltzmann probabilities or free energies.
- **There is no confidence score** of the pLDDT or PAE kind on the samples.
- **Slow processes are hypotheses.** Nearly all of the dynamics training data is MD of up to 1 µs. The paper says rates and pathways of binding, dissociation, folding and other slow rearrangements need independent validation.
- **Each propagation step is an independent endpoint** at the chosen lag, not a frame of a continuous trajectory. The documented lag range is 1 to 1000 ns. Longer lags tend towards the equilibrium ensemble.
- **A complex can fail without any sign in the structures.** Every sample can pass the physicality checks and have the chains in contact, while the partner sits somewhere different in each one. Step 8 measures this, and the 2A9K example below shows it.

Generated structures are computational hypotheses and should be validated before they inform anything downstream.

## The examples and their MD reference
The notebook's first example is a complex, HPr kinase with HPr ([1KKL](https://www.rcsb.org/structure/1KKL)): three copies of the kinase plus one HPr, 614 residues. A single protein is what BioEmu2's own [examples](https://github.com/microsoft/bioemu2/tree/main/examples) already show, so this notebook spends its first example on the case that is harder and more interesting, a protein–protein interface.

Section 8 then asks whether that interface is *consistent* across the ensemble, rather than only showing it: an ensemble can have contact in every sample and still be telling you nothing, if the chains meet somewhere different each time. It also draws the ensemble next to molecular dynamics of the same complex, 21 frames per example from the [DynaBench](https://doi.org/10.1016/j.jmb.2026.169650) dataset of three 100 ns replicas per complex of Docking Benchmark 5.5. Those frames live in `md_reference/` and are **not** covered by the MIT licence of this repository; if you use them, cite Barlas AB, Laurent B, Karaca E, Prévost C, Sacquin-Mora S, *DynaBench: Dynamic data for the docking benchmark*, Journal of Molecular Biology 438 (2026) 169650, [doi:10.1016/j.jmb.2026.169650](https://doi.org/10.1016/j.jmb.2026.169650). The full trajectories are at [www-lbt.ibpc.fr/DynaBench](http://www-lbt.ibpc.fr/DynaBench).

The second example is one BioEmu2 does **not** resolve: RalA with the C3bot1 exoenzyme of *Clostridium botulinum* ([2A9K](https://www.rcsb.org/structure/2A9K)), 377 residues. It is a host–pathogen pair, so few paired sequences exist for the interface, and AlphaFold-Multimer misses it as well (ipTM 0.28). Measured the same way, with the same DynaBench frames for reference:

| | samples or frames in contact | distinct residue pairs | highest frequency | pairs in ≥ half |
|---|---|---|---|---|
| 1KKL, DynaBench MD (kinase copy 3 · HPr) | 21/21 | 94 | 1.00 | 14 |
| 1KKL, BioEmu2, 50 samples | 40/50 | 257 | 0.52 | 2 |
| 2A9K, DynaBench MD (RalA · C3bot1) | 21/21 | 86 | 1.00 | 63 |
| 2A9K, BioEmu2, 50 samples | 50/50 | 1134 | 0.12 | 0 |

In 2A9K every BioEmu2 sample is in contact and every one passes the physicality checks, yet no residue pair is shared by more than 6 of the 50: the partner is placed somewhere different each time. Looking only at the structures, or at physicality, would not reveal it.

Step 8's viewer shows the same thing in space: superposed on RalA, C3bot1 sits a median 54 Å from where the first sample put it, against 3 Å across the MD frames. If you already know one contact of such an interface, contact conditioning (see above) is the tool BioEmu2 offers for it; the notebook does not use it.

## Benchmark data

[`benchmark_data/`](benchmark_data) holds the tables of our benchmark of BioEmu2 on DynaBench: for 15 interfaces of 13 Docking Benchmark 5.5 complexes, the inter-protein contact frequencies in MD, in five AlphaFold 2.3 models and in 50 BioEmu2 samples, with per-interface summaries, per-residue surface values and the sequences used. Its [README](benchmark_data/README.md) gives the definitions and the columns. The MD-derived values come from DynaBench; please cite it if you use them.

## Credits and licence

BioEmu2 is developed by Microsoft Research and released under the MIT licence; its weights are published under the MIT licence on [Hugging Face](https://huggingface.co/microsoft/bioemu2). This repository contains neither: the notebook installs BioEmu2 from its own GitHub repository (pinned to commit `6d1ca74`, 3 November 2026) and downloads the weights at run time.

Portions of the notebook are adapted from the examples in microsoft/bioemu2, which carry the following notice, repeated in [NOTICE](NOTICE):

> Copyright (c) Microsoft Corporation. Licensed under the MIT License.

The notebook itself is released under the MIT licence, see [LICENSE](LICENSE).

## Citation

If you use BioEmu2, cite the BioEmu2 paper, not this notebook:

> Gastegger M, Foster AE, Jiménez-Luna J, Hempel T, Lewis S, Xie Y, Sun L, Zaporozhets I, Frank JT, Chen Y, Plainer M, Winkler L, Reis PBPS, Giese SH, Sordillo R, García Satorras V, Inanc E, Kapuśniak K, Yang S, Raghu R, Kim R, Dernbach G, Gunceler D, Schulz H, Kruft B, Segler M, Bishop CM, Clementi C, Tomioka R, Noé F. BioEmu2: Scalable emulation of biomolecular dynamics. arXiv:2611.27924 (2026). [doi:10.48550/arXiv.2611.27924](https://doi.org/10.48550/arXiv.2611.27924)

```bibtex
@article{gastegger2026bioemu2,
  title   = {{BioEmu2}: Scalable Emulation of Biomolecular Dynamics},
  author  = {Gastegger, Michael and Foster, Adam E. and Jim{\'e}nez-Luna, Jos{\'e} and Hempel, Tim and Lewis, Sarah and Xie, Yu and Sun, Lixin and Zaporozhets, Iryna and Frank, J. Thorben and Chen, Yaoyi and Plainer, Michael and Winkler, Ludwig and Reis, Pedro B. P. S. and Giese, Sven H. and Sordillo, Roberto and Garc{\'i}a Satorras, Victor and Inanc, Eray and Kapu{\'s}niak, Kacper and Yang, Soojung and Raghu, Rishwanth and Kim, Rachel and Dernbach, Gabriel and Gunceler, Deniz and Schulz, Hannes and Kruft, Bonnie and Segler, Marwin and Bishop, Christopher M. and Clementi, Cecilia and Tomioka, Ryota and No{\'e}, Frank},
  journal = {arXiv preprint arXiv:2611.27924},
  year    = {2026},
  doi     = {10.48550/arXiv.2611.27924}
}
```

If you use the MD frames in `md_reference/`, cite DynaBench as well (see above).

---

Maintained by the [Computational Structural Biology Lab](https://csb-karacalab.github.io/), Izmir Biomedicine and Genome Center.
