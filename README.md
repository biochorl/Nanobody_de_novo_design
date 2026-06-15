<img src="Images/Background.png" alt="background" width="100%"/>

[Background image source](https://www.independent.co.uk/travel/europe/slovenia/nova-gorica-slovenia-italy-capital-of-culture-b2712344.html)

# *De Novo* Nanobody Design

> 🌐 **Interactive website:** [biochorl.github.io/Nanobody_de_novo_design](https://biochorl.github.io/Nanobody_de_novo_design/) — the same workflow with a guided, mobile-friendly flow and one-click "Open in Colab" buttons.

This repository contains the computational workflow for the *de novo* (re)design of a nanobody to a specific target protein of interest.
The pipeline is developed to use Google Colab resources and is adequate for people with little knowledge of protein *de novo* design tools and limited structural biology background.
It is made for demonstration and teaching, while being **not** adequate to scale for a production-level *in silico* screening.

This material has been used during the **Nanobody Workshop (22-26 Sep 2025)** at the **University of Nova Gorica**, Rozna Dolina campus.
*   **Event Link:** [https://indico.ijs.si/event/2966/](https://indico.ijs.si/event/2966/)
*   A brief introduction on AI-assisted protein design is available at this [link](./Misc/De-novo_design_intro.pdf)

## Overview

The workflow starts from the 3D structure of two input PDBs — one of the **target** antigen and one of the **scaffold nanobody** — and produces, in **four Google Colab notebooks**, a full-atom *de novo* nanobody–antigen complex with an orthogonal confidence check:

| # | Notebook | What it does | Engine |
|---|----------|--------------|--------|
| 1 | **Preparation** | Define the epitope on the antigen and mark the nanobody CDRs to redesign | DiscoTope-3.0 + Nanocdr-X |
| 2 | **IgGM Design & PIPPack** | Co-design CDR sequence + structure against the antigen, then repack side chains | IgGM + PIPPack |
| 3 | **AntiFold Redesign** | Optimize the CDR sequences in the context of the complex | AntiFold |
| 4 | **ESMFold Validation** | Blindly re-fold the designed sequence and align it to the design | ESMFold2 (BioHub) |

This 4-step pipeline replaces the previous 6-step one (RFantibody → ProteinMPNN → PIPPack → AntiFold → gapTrick). The old notebooks are kept in [`backup_old_pipeline/`](./backup_old_pipeline).

**Note on File Access:** The links to files in this README (like [7z1b.pdb](./Example_input/7z1b.pdb)) are relative paths. If you click on them in a browser while logged into GitHub with access to this repository, you will see the file's content. To use the file, you will need to manually click the "Download raw file" <img src="Images/Download_symbol.png" alt="Download button" width="200"/> top-right button on the file viewer page.

> 📱 **On a phone / no laptop?** Every notebook has a **"Mobile mode"** switch at the top: turn it on and the required input files are downloaded automatically from this repository, so you can run the whole workflow from Google Colab without ever touching a local file system.

## Target and nanobody scaffold files (initial inputs)

*   **Target:** An arbitrary protein structure (in PDB format) of your choice.
    *   **Example Target (SARS-CoV-2 RBD):** [7z1b.pdb](./Example_input/7z1b.pdb)
*   **Scaffold:** A pre-selected nanobody structure to serve as the starting point for the (re)design.
    *   **Example nanobody scaffold:** [nanobody_scaffold.pdb](./Example_input/nanobody_scaffold.pdb)

---
### 1. Candidate epitope prediction & nanobody CDR identification

<table>
  <tr>
    <td align="center">
      <img src="Images/Image_target_patches_detection.png" alt="Predicting target antigen epitopes" width="400"/>
      <br>
      <em>Predicting epitope hotspots on the target antigen</em>
    </td>
    <td align="center">
      <img src="Images/Image_CDRs_detection.png" alt="Identifying nanobody CDRs" width="400"/>
      <br>
      <em>Identifying the nanobody complementarity-determining regions (CDRs)</em>
    </td>
  </tr>
</table>

*   **Tools:** [DiscoTope-3.0](https://services.healthtech.dtu.dk/services/DiscoTope-3.0/) and [Nanocdr-X](https://github.com/lescailab/nanocdr-x)
*   **Purpose:** This step prepares everything IgGM needs:
    *   It predicts **B-cell epitope hotspots** on the surface of the target antigen with DiscoTope-3.0 and clusters them into patches you can target (output as **0-based positional indices** for IgGM).
    *   It identifies and masks the **CDRs** of the nanobody scaffold, building the `IgGM_input.fasta` (nanobody framework fixed, CDRs marked as `X`) and the single-chain `antigen_A.pdb`.
*   **Colab Notebook:** [![1_Preparation_colab.ipynb](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/biochorl/Nanobody_de_novo_design/blob/main/1_Preparation_colab.ipynb)
*   **Example output:** epitope/CDR annotations plus the IgGM-ready inputs.
    *   **Annotation file:** [Step_1_annotations.txt](./Example_output/Step_1_annotations.txt)
    *   **IgGM input FASTA (facsimile):** [IgGM_input.fasta](./Example_output/IgGM_input.fasta)
    *   **Single-chain antigen PDB (facsimile):** [antigen_A.pdb](./Example_output/antigen_A.pdb)
---
### 2. *De novo* CDR design & side-chain refinement (IgGM + PIPPack)

<table>
  <tr>
    <td align="center">
      <img src="Images/RFDiffusion_sample_image.gif" alt="Generative antibody design" width="430"/>
      <br>
      <em>Generative co-design of sequence and structure</em>
    </td>
    <td align="center">
      <img src="Images/pippack_architecture.png" alt="Side-chain packing" width="430"/>
      <br>
      <em>Side-chain repacking (PIPPack)</em>
    </td>
  </tr>
</table>

*   **Tools:** [IgGM](https://www.biorxiv.org/content/10.1101/2024.09.19.613838v2) and [PIPPack](https://onlinelibrary.wiley.com/doi/10.1002/prot.26705)
*   **Purpose:** A single notebook that replaces the old **RFantibody + ProteinMPNN + PIPPack** sequence:
    *   **IgGM** co-designs the **sequence and structure** of the nanobody CDRs against the antigen, producing a **full all-atom** nanobody–antigen complex (no glycine masking, side chains completed via PDBFixer).
    *   You can optionally **sample a range of CDR3 lengths** — each design gets an independently sampled CDR3 length (typical nanobody range).
    *   **PIPPack** then repacks the side chains of the new CDRs *in the context of the antigen*, and the notebook exports a `cdrs_annotation.json` carrying the exact 0-based CDR mask of every design for the next step.
*   **Colab Notebook:** [![2_IgGM_colab.ipynb](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/biochorl/Nanobody_de_novo_design/blob/main/2_IgGM_colab.ipynb)
*   **Example output:** the refined full-atom nanobody–antigen complex(es) and the CDR mask used downstream.
    *   **Refined design PDB (facsimile):** [Nanobody_Design_0.pdb](./Example_output/Nanobody_Design_0.pdb)
    *   **CDR annotation (facsimile):** [cdrs_annotation.json](./Example_output/cdrs_annotation.json)
---
### 3. CDR sequence optimization in complex context (AntiFold)

<table>
  <tr>
    <td align="center">
      <img src="Images/AntiFold.jpeg" alt="CDR sequence optimization" width="600"/>
      <br>
      <em>CDR sequence optimization with a CDR-specialized inverse-folding model</em>
    </td>
  </tr>
</table>

[image source](https://doi.org/10.1093/bioadv/vbae202)

*   **Tool:** [AntiFold](https://academic.oup.com/bioinformaticsadvances/article/5/1/vbae202/8090019)
*   **Purpose:** To re-design the **CDR sequences** of the IgGM nanobody with a CDR-specialized model, *in the context of the full refined complex* (the antigen is frozen as structural context). The notebook reads the per-design `cdrs_annotation.json`, so it masks **only** the variable-length CDR residues — no static-FASTA misalignment.
*   **Colab Notebook:** [![3_Antifold_colab.ipynb](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/biochorl/Nanobody_de_novo_design/blob/main/3_Antifold_colab.ipynb)
*   **Example output:** the top redesigned nanobody sequence (lower score is better here), packaged with the complex PDBs.
    *   **Best design FASTA:** [design_3_score_0.4623.fasta](./Example_output/design_3_score_0.4623.fasta)
---
### 4. Blind validation by structure prediction (ESMFold2)

<table>
  <tr>
    <td align="center">
      <img src="Images/GapTrick.png" alt="Blind folding validation" width="600"/>
      <br>
      <em>Folding the designed sequence from scratch and aligning it to the design</em>
    </td>
  </tr>
</table>

*   **Tool:** [ESMFold2](https://www.biohub.ai/models/esmfold2) via the BioHub API
*   **Purpose:** An independent, orthogonal check. ESMFold2 folds the **entire AntiFold-redesigned complex sequence *ab initio*** (purely from sequence, *without* using the PIPPack coordinates as a starting guess). The predicted structure is rigidly aligned to the design to compute the RMSD, and the notebook reports the **pLDDT, pTM and ipTM** confidence metrics — ipTM being the key metric for the nanobody–antigen interface.
*   **Colab Notebook:** [![4_ESMFold_colab.ipynb](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/biochorl/Nanobody_de_novo_design/blob/main/4_ESMFold_colab.ipynb)
*   **Example output:** the blindly folded complex and its confidence metrics.
    *   **Repredicted structure (facsimile):** [Nanobody_Design_0_esmfold2.cif](./Example_output/Nanobody_Design_0_esmfold2.cif)
    *   **Confidence metrics (facsimile):** [Nanobody_Design_0_metrics.json](./Example_output/Nanobody_Design_0_metrics.json)

---
## Screening and Further Validation of *de novo* designed nanobody binders

Further steps of validation are crucial for increasing the likelihood of success in experimental settings, both *in vitro* and *in vivo*.
In order to have a sufficient n° of designs for a successful experimental screening, it is advisable to generate at least **≈1000** *in silico* designs (full-atom structure) and use them as input to pass further validation and filtering steps, as these will likely result in 95% of designs to fail leaving your computer.
These steps may use one or more of the following (the list of options for each approach is not comprehensive, it is just for giving some examples):

*   **Interaction confidence from deep learning approaches**
    *   Use AlphaFold2 (AF2), AlphaFold-Multimer or AF3 to repredict the complex without the use of templates and evaluate the models for consistency with the design structure and flexibility-sensitive confidence scores (e.g, [Local Interaction Score](https://github.com/flyark/AFM-LIS) or [ipSAE](https://doi.org/10.1101/2025.02.10.637595))
*   **Empirical-physics scores:**
    *   Protein-protein docking scores (e.g., HADDOCK, Rosetta-ddG, Fold-X)
    *   Geometric scores using surface and non-covalent interactions features at interface (e.g., Rosetta interface analysis tools, dG/dSASA, packstat or n° unsaturated H-bonds)
*   **Physics-based Assessment:**
    *   Molecular Dynamics simulations (> 100 ns) to assess complex stability (e.g., RMSD metric using GROMACS, AMBER, etc...)
    *   Enhanced sampling methods, to generate the conformational free-energy landscape (e.g., metadynamics with PLUMED or coarse-grained/all-atom hybrid simulations with Prody)
*   **Developability Assessment (To be compared with the initial Nb scaffold if it is known it has a good developability):**
    *   Solubility prediction
    *   Stability Prediction
    *   Aggregation propensity
    *   Propension for off-target antigenic interactions

---
## Acknowledgements

Most of the Google colaboratories were developed by customizing other colabs, referenced in the corresponding file. I linked the paper or the original source describing all the different methods.

## Contact

For any questions, suggestions, or issues, please open an issue in this GitHub repository or contact me at [marco.orlando1991@live.it](mailto:marco.orlando1991@live.it) or [marco.orlando@ung.si](mailto:marco.orlando@ung.si).
