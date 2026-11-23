# Benchmark data: BioEmu2 on DynaBench

Data behind the preprint *Benchmarking BioEmu2 on DynaBench: a first independent test of sampling protein
interface dynamics*. For 15 interfaces of 13 Docking Benchmark 5.5 complexes, it compares three sources of
inter-protein residue contacts:

- **MD**: three 100 ns replicas per complex from [DynaBench](https://doi.org/10.1016/j.jmb.2026.169650)
  (CHARMM36m), first 20 ns of each replica discarded;
- **AlphaFold 2.3 baseline**: AlphaFold2-Multimer v3 through ColabFold 1.6.3, five models, no templates;
- **BioEmu2**: 50 equilibrium samples per complex ([microsoft/bioemu2](https://github.com/microsoft/bioemu2),
  commit `6d1ca74`).

This folder holds the tables. The structures themselves (BioEmu2 ensembles and AlphaFold models) are larger and
are archived separately; the link will be added here. The MD trajectories are not redistributed: they are at
[www-lbt.ibpc.fr/DynaBench](https://www-lbt.ibpc.fr/DynaBench).

## Definitions

- **Contact**: two residues of different proteins with any two heavy atoms within 4.5 Å.
- **Contact frequency**: fraction of MD frames (mean of the three replicas), of the five AlphaFold models, or
  of the 50 BioEmu2 samples in which the pair is in contact.
- **Persistent / predicted**: MD frequency ≥ 0.5; AlphaFold ≥ 0.6 (3 of 5 models); BioEmu2 ≥ 0.5.
- **Recovery**: share of the MD-persistent pairs that a model also predicts. **Precision**: share of a model's
  predicted pairs that are MD-persistent.
- **ρ**: Spearman correlation of frequencies over interface-shell pairs (residues within 10 Å of the partner
  in ≥ 10% of MD frames) that are in contact at least once in either of the two methods compared.
- **MD replica values**: each replica compared with the mean of the other two, averaged over the three.
- **Footprint**: residues whose highest contact frequency with any partner residue is ≥ 0.02.
- **Paired-alignment depth**: N_eff / √L of the ColabFold paired alignment of the two construct sequences
  (rows covering ≥ 50% of each partner, weights at 80% identity, L = combined length).

## Files

| File | Content |
|---|---|
| `interfaces.csv` | One row per interface: proteins, alignment depth, AlphaFold ipTM, counts of input, MD-persistent and predicted contacts, recovery, precision and ρ for both models, the MD replica values, and footprint sizes. Two further complexes without BioEmu2 samples (1ZM4, 4LW4; out of GPU memory) are listed with `has_bioemu2` = False. |
| `contact_frequencies/<PDB>_<pair>.csv` | One row per residue pair with a contact in any source: residue numbers and types, MD frequency (mean and each replica), AlphaFold and BioEmu2 frequencies, whether the pair is a contact in the MD input (crystal) structure, and whether it lies in the interface shell. |
| `residue_surfaces/<PDB>_<pair>.csv` | One row per residue of the two proteins: its highest contact frequency with any partner residue in MD, AlphaFold and BioEmu2 (the values colored on the surface figures). |
| `alphafold_models.csv` | ipTM of each of the five ranked AlphaFold models per complex. |
| `bioemu2_samples.csv` | Per BioEmu2 sample: physicality check, fraction of input-structure inter-protein contacts kept (`fnat`), and Cα RMSD to the MD input structure. Not computed for 3K75. |
| `sequences.fasta` | The construct sequences given to AlphaFold and BioEmu2 (those of the MD systems). |

## Notes

- **Naming.** `<pair>` gives the two proteins of the MD system, for example `A-B`. In 1KKL, `C` is HPr kinase
  (three copies, merged by taking the highest frequency) and `D` is HPr. 2MTA has three interfaces.
- **2O8V.** DynaBench lists PAPS reductase–thioredoxin under the label 2OV8; its PDB ID is 2O8V, used here.
  `interfaces.csv` keeps the DynaBench label in `dynabench_label`.
- **Residue numbers** in these tables follow the MD systems (PDB numbering). The AlphaFold and BioEmu2
  structure files number each chain from 1, in the order of `sequences.fasta`.
- **Paired alignments** are not included (about 100 MB). They can be rebuilt from `sequences.fasta` with the
  ColabFold MMseqs2 server.

## Citation and reuse

If you use these data, please cite the preprint (reference to be added on posting) and, for anything derived
from the MD, DynaBench: Barlas AB, Laurent B, Karaca E, Prévost C, Sacquin-Mora S. *DynaBench: dynamic data for
the docking benchmark.* J Mol Biol 438, 169650 (2026), doi:10.1016/j.jmb.2026.169650. BioEmu2 should be cited as
Gastegger M, et al., arXiv:2611.27924 (2026).
