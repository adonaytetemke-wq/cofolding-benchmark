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

## Limitations
Predictions are apo (no ligand or heme) and compared against single crystal snapshots. TM-score and GDT are approximate.

## Tools
AlphaFold Database, Biopython, py3Dmol. Structures from the RCSB Protein Data Bank: COX-1 (1EQG), COX-2 (3LN1).

## License
MIT
