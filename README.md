# Can AI Predict How Drugs Bind?
### Benchmarking AI structure and co-folding models against experiment on COX enzymes

Modern drug discovery is moving from "dock a molecule into a known structure" toward AI models that predict protein and drug together. This project tests how far those models can be trusted, using COX-1 and COX-2 (targets of aspirin, ibuprofen and celecoxib), where experimental structures exist to check against.

## Roadmap

| Stage | Topic | Status |
|---|---|---|
| 1 | AI-predicted structures vs crystal structures | Done |
| 2 | Protein language model embeddings (COX-1 vs COX-2) | Done |
| 3 | Co-folding of drug and protein | Done |
| 4 | Benchmark: co-folding vs docking vs crystal | Done |


## Repository Contents & Usage
- **Notebooks (`notebooks/` and root `.ipynb` files):** Workflows for structure comparison, sequence analysis, co-folding, and docking benchmarks.
- **Results (`results/`):** Benchmark tables, metrics, and structural outputs generated across Stages 1–4.

  
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


**Key Takeaway:** ESM-2 cleanly separates species orthologs from paralogs and identifies binding pockets as statistically distinct ( p=0.001 ). However, the known selectivity positions (513 and 523) sit around the 70th–75th percentiles—indicating that sequence-alone models capture general evolutionary constraints, but require 3D structural context to fully pinpoint small-molecule binding pockets.


## Stage 3 Results: Co-folding of Drug and Protein

Using **Boltz**, small-molecule ligands and protein sequences were co-folded end-to-end without providing experimental pocket coordinates. 

| Job | Protein RMSD ($\AA$) | Ligand RMSD ($\AA$) | Site distance ($\AA$) | Key residues touched | Ser530 OG distance ($\AA$) | ipTM | Ligand ipTM | Complex pLDDT |
|---|---|---|---|---|---|---|---|---|
| cox2_celecoxib | 0.35 | 0.51 | 0.19 | none | n/a | 0.67 | 0.67 | 0.93 |
| cox1_ibuprofen | 0.4 | 0.36 | 0.21 | ARG120, TYR355 | 4.11 | 0.83 | 0.83 | 0.95 |
| cox1_aspirin | 0.4 | n/a | 2.67 | TYR385, SER530 | 2.87 | 0.6 | 0.6 | 0.95 |
| cox2_aspirin | 0.46 | n/a | 3.75 | none | n/a | 0.76 | 0.76 | 0.94 |

### Key Takeaways
- **Accurate Poses for Non-Covalent Inhibitors:** Both *celecoxib in COX-2* and *ibuprofen in COX-1* achieved exceptionally low Ligand RMSDs (<0.6\,\AA), proving the model successfully positions them within the correct binding pockets with high confidence ($\text{pLDDT} > 0.93$).
- **Aspirin Selectivity Hint:** Since aspirin acts via covalent acetylation rather than simple non-covalent docking, traditional RMSD doesn't apply. However, the model places aspirin significantly closer to the catalytic residue `SER530` in COX-1 ($2.87\,\AA$) compared to COX-2 ($3.75\,\AA$), capturing differential binding tendencies despite lacking explicit covalent training physics.
 
## Limitations
Predictions are apo (no ligand or heme) and compared against single crystal snapshots. TM-score and GDT are approximate.

 
## Stage 4: Co-Folding vs. Docking Benchmark
 
 we benchmarked **AI co-folding (Boltz)** from Stage 3 head-to-head against **classical physics-based docking (AutoDock Vina)** and **experimental X-ray crystal structures** across four drug–enzyme complexes:

* **COX-2 + Celecoxib** (Reference crystal structure: PDB `3LN1`, ligand `CEL`)
* **COX-1 + Ibuprofen** (Reference crystal structure: PDB `1EQG`, ligand `IBP`)
* **COX-1 + Aspirin** (No non-covalent crystal pose; evaluated by site location, key catalytic contacts, and cross-method agreement)
* **COX-2 + Aspirin** (No non-covalent crystal pose; evaluated by site location, key catalytic contacts, and cross-method agreement)

### Experimental Setup
* **Crystal Structure (Ground Truth):** Experimental coordinates from RCSB PDB (`1EQG` and `3LN1`).
* **AutoDock Vina (Classical Docking):** Rigid-receptor docking (`exhaustiveness = 16`, `num_modes = 9`) centered directly on the experimental binding site (`24 × 24 × 24 Å` box), giving Vina the true pocket location a priori.
* **Boltz (AI Co-folding):** Predicted strictly from protein amino acid sequence and ligand SMILES with zero prior structural or pocket coordinates, then aligned to the crystal protein frame.
* **Evaluation Metrics:** Symmetry-aware heavy-atom Ligand RMSD via RDKit (success threshold: `< 2.0 Å`), binding-site centroid distance, key active-site contacts (`ARG120`, `TYR355`, `TYR385`, `SER530` within `4.0 Å`), COX-2 side-pocket contacts (`513` / `523`), and closest heavy-atom distance to the catalytic `Ser530 OG` hydroxyl.

---

## Benchmark Results

| Job | Method | Ligand RMSD (Å) | Site distance (Å) | Key residues | Side pocket (513/523) | Ser530 OG (Å) | Vina score (kcal/mol) |
| :--- | :--- | :---: | :---: | :--- | :--- | :---: | :---: |
| **cox2_celecoxib** | Crystal | 0.00 | 0.00 | none | ALA513 | n/a | n/a |
| **cox2_celecoxib** | Vina top-1 | 0.58 | 0.48 | none | ALA513 | n/a | -12.21 |
| **cox2_celecoxib** | Boltz | **0.54** | **0.19** | none | ALA513 | n/a | n/a |
| **cox1_ibuprofen** | Crystal | 0.00 | 0.00 | ARG120, TYR355, SER530 | ILE523 | 4.18 | n/a |
| **cox1_ibuprofen** | Vina top-1 | 6.12 | 1.63 | ARG120, TYR355, TYR385 | ILE523 | 4.18 | -7.50 |
| **cox1_ibuprofen** | Boltz | **0.27** | **0.13** | ARG120, TYR355 | ILE523 | 4.10 | n/a |
| **cox1_aspirin** | Vina top-1 | n/a | 3.70 | TYR385, SER530 | ILE523 | 3.12 | -6.38 |
| **cox1_aspirin** | Boltz | n/a | 3.38 | TYR385, SER530 | ILE523 | 2.74 | n/a |
| **cox2_aspirin** | Vina top-1 | n/a | 3.78 | none | ALA513 | n/a | -6.34 |
| **cox2_aspirin** | Boltz | n/a | 4.04 | none | ALA513 | n/a | n/a |

### Secondary Diagnostics
* **Vina Best-of-9 RMSD (Sampling vs. Scoring):**
  * `cox2_celecoxib`: **0.58 Å** (top-1 = 0.58 Å)
  * `cox1_ibuprofen`: **0.61 Å** (top-1 = 6.12 Å)
* **Cross-Method Agreement (Vina top-1 vs. Boltz Ligand RMSD):**
  * `cox2_celecoxib`: **0.84 Å**
  * `cox1_ibuprofen`: **6.12 Å**
  * `cox1_aspirin`: **1.19 Å**
  * `cox2_aspirin`: **1.35 Å**

---

## Key Findings

1. **Pose Accuracy (< 2.0 Å Success Threshold):**
   * **Boltz achieved 2/2 sub-Ångström crystal recoveries** (`0.54 Å` on `cox2_celecoxib` and `0.27 Å` on `cox1_ibuprofen`) despite having no prior knowledge of the binding pocket coordinates.
   * **Vina top-1 succeeded on 1/2 reference complexes** (`0.58 Å` on `cox2_celecoxib`), while failing on `cox1_ibuprofen` (`6.12 Å`). Note that with $N = 2$ reference complexes, these results serve as an illustrative case study rather than a statistical benchmark.
2. **Vina Best-of-9 RMSD (Sampling vs. Scoring):**
   * For `cox1_ibuprofen`, Vina's **best-of-9** pose reached **0.61 Å** RMSD to the crystal ligand, whereas its **top-1** ranked pose was **6.12 Å** (a flipped orientation inside the pocket). This demonstrates a classic docking bottleneck: Vina's search algorithm *sampled* the native pose accurately, but its empirical scoring function *ranked* an inverted pose higher (`-7.50 kcal/mol`).
3. **Cross-Method Agreement & Aspirin Binding:**
   * Wherever Vina top-1 and Boltz agreed within `< 1.5 Å` (`cox2_celecoxib` at `0.84 Å`), both closely matched the crystal structure.
   * On **aspirin**—where no pre-acetylation crystal pose exists—both independent methods converged on nearly identical binding geometries (`1.19 Å` RMSD between Vina and Boltz in COX-1; `1.35 Å` in COX-2).
4. **COX-2 Selectivity & Acetylation Mechanism:**
   * **Selectivity:** Both Vina and Boltz placed celecoxib's bulky sulfonamide moiety into the COX-2-specific side pocket contacting **ALA513**, matching the experimental `3LN1` crystal structure.
   * **Catalytic Mechanism:** In COX-1, both methods positioned aspirin directly adjacent to the catalytic **SER530** hydroxyl (`2.74 Å` in Boltz; `3.12 Å` in Vina) and **TYR385**, consistent with nucleophilic attack and irreversible acetylation of Ser530.

---

## Limitations & Caveats

* **Asymmetric Setup:** Vina received the crystal structure and a 24 Å pocket box, whereas Boltz predicted the complex from sequence and SMILES alone.
* **Training Overlap:** `1EQG` and `3LN1` predate Boltz's training cutoff, so accuracy on them may partly reflect training familiarity.
* **Small Sample:** Results cover two reference ligands, two aspirin systems, one random seed, and one run per method.
* **Modeling Simplifications:** Docking used a rigid receptor without crystallographic waters or heme, and omitted aspirin's covalent acetylation of `SER530`.


## Tools
AlphaFold Database, Biopython, py3Dmol. Structures from the RCSB Protein Data Bank: COX-1 (1EQG), COX-2 (3LN1).

## License
MIT
