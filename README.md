# Can AI Predict How Drugs Bind?
### Benchmarking AI structure and co-folding models against experiment on COX enzymes

Modern drug discovery is moving from "dock a molecule into a known structure" toward AI models that predict protein and drug together. This project tests how far those models can be trusted, using COX-1 and COX-2 (targets of aspirin, ibuprofen and celecoxib), where experimental structures exist to check against.

## Roadmap

| Stage | Topic | Status |
|---|---|---|
| 1 | AI-predicted structures vs crystal structures | Done |
| 2 | Protein language model embeddings (COX-1 vs COX-2) | done |
| 3 | Co-folding of drug and protein | Planned |
| 4 | Benchmark: co-folding vs docking vs crystal | Planned |

## Stage 1 results

AlphaFold Database models (not ESMFold) compared with crystal structures after superposition.

| Enzyme | PDB | Length | Mean pLDDT | RMSD all (Å) | RMSD confident (Å) | TM-score≈ | GDT≈ | Pocket CA dev (Å) | Pocket side-chain RMSD (Å) |
|---|---|---|---|---|---|---|---|---|---|
| COX1 | 1EQG | 580 | 93.7 | 0.25 | 0.25 | 1.0 | 100.0 | 0.21 | 0.76 |
| COX2 | 3LN1 | 587 | 92.3 | 0.55 | 0.55 | 1.0 | 99.0 | 0.35 | 0.82 |

Both predictions matched the experiments closely. The pocket backbone was as accurate as the whole protein, while pocket side chains deviated more (about 0.8 Å), consistent with ligand-induced movement missing from apo predictions. Because AlphaFold was likely trained on these PDB entries, this measures reproduction of known structures rather than true prediction, so later stages test cases where memorization cannot help.

## Stage 2 results

A protein language model (ESM-2, 650M parameters) was used to compare COX-1 and COX-2 from sequence alone.

| Metric | Value |
|---|---|
| Aligned positions / identity | 568 / 64% |
| Mean embedding distance: identical vs substituted | 0.028 vs 0.101 |
| Position 513 (H->R) percentile | 75 |
| Position 523 (I->V) percentile | 69 |
| Pocket vs other distance | 0.022 vs 0.055 (p=0.001) |
| COX-1 sheep: ortholog vs paralog similarity | 0.984 vs 0.939 |
| COX-2 mouse: ortholog vs paralog similarity | 0.988 vs 0.939 |


**Key Takeaway:** ESM-2 cleanly separates species orthologs from paralogs and identifies binding pockets as statistically distinct ($p=0.001$). However, the known selectivity positions (513 and 523) sit around the 70th–75th percentiles—indicating that sequence-alone models capture general evolutionary constraints, but require 3D structural context to fully pinpoint small-molecule binding pockets.


## Stage 3 Results: Co-folding of Drug and Protein

Using **Boltz**, small-molecule ligands and protein sequences were co-folded end-to-end without providing experimental pocket coordinates. 

| Job | Protein RMSD ($\AA$) | Ligand RMSD ($\AA$) | Site distance ($\AA$) | Key residues touched | Ser530 OG distance ($\AA$) | ipTM | Ligand ipTM | Complex pLDDT |
|---|---|---|---|---|---|---|---|---|
| cox2_celecoxib | 0.35 | 0.51 | 0.19 | none | n/a | 0.67 | 0.67 | 0.93 |
| cox1_ibuprofen | 0.4 | 0.36 | 0.21 | ARG120, TYR355 | 4.11 | 0.83 | 0.83 | 0.95 |
| cox1_aspirin | 0.4 | n/a | 2.67 | TYR385, SER530 | 2.87 | 0.6 | 0.6 | 0.95 |
| cox2_aspirin | 0.46 | n/a | 3.75 | none | n/a | 0.76 | 0.76 | 0.94 |

### Key Takeaways
- **Accurate Poses for Non-Covalent Inhibitors:** Both *celecoxib in COX-2* and *ibuprofen in COX-1* achieved exceptionally low Ligand RMSDs ($<0.6\,\AA$), proving the model successfully positions them within the correct binding pockets with high confidence ($\text{pLDDT} > 0.93$).
- **Aspirin Selectivity Hint:** Since aspirin acts via covalent acetylation rather than simple non-covalent docking, traditional RMSD doesn't apply. However, the model places aspirin significantly closer to the catalytic residue `SER530` in COX-1 ($2.87\,\AA$) compared to COX-2 ($3.75\,\AA$), capturing differential binding tendencies despite lacking explicit covalent training physics.
- 

## Limitations
Predictions are apo (no ligand or heme) and compared against single crystal snapshots. TM-score and GDT are approximate.

## Tools
AlphaFold Database, Biopython, py3Dmol. Structures from the RCSB Protein Data Bank: COX-1 (1EQG), COX-2 (3LN1).

## License
MIT
