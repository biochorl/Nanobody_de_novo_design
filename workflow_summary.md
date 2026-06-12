# IgGM + PIPPack + AntiFold + ESMFold: New Nanobody Design Workflow

This document summarizes the newly consolidated 4-step workflow for *de novo* nanobody design, structural refinement, and sequence optimization.

## Notebook 1: Preparation (`1_Preparation_colab.ipynb`)
- **Objective:** Prepare the antigen structure and define the initial sequence scaffolding.
- **Key Actions:** 
  - Trims the antigen PDB.
  - Generates the `IgGM_input.fasta` containing the fixed framework sequences and the CDRs marked as `X`.
  - Automatically identifies the binding epitope hotspots using DiscoTope-3.

## Notebook 2: IgGM Design & PIPPack Refinement (`2_IgGM_colab.ipynb`)
*This single notebook replaces the old RFantibody + ProteinMPNN + PIPPack multi-step process.*
- **Objective:** Generate *de novo* sequence/structure pairs for the nanobody CDRs and rebuild the side chains.
- **Key Actions:**
  - Uses **IgGM** to co-design the CDR sequences and backbone coordinates against the antigen.
  - Dynamically samples varying lengths for CDR3 if enabled.
  - Automatically invokes **PDBFixer** internally to construct missing full-atom backbone geometries (particularly Oxygen atoms) required by PIPPack.
  - Uses **PIPPack** to repack and refine the side chains of the new CDR sequences onto the IgGM backbone.
  - Dynamically calculates the structural lengths of the generated nanobodies and exports `cdrs_annotation.json` into the output ZIP, providing perfect 0-based masking indices for downstream redesign.

## Notebook 3: AntiFold Redesign (`3_Antifold_colab.ipynb`)
- **Objective:** Optimize the IgGM-designed nanobody sequences by redesigning the CDRs in the context of the full refined complex.
- **Key Actions:**
  - Takes the full `PIPPack_outputs.zip` containing the complex PDBs and the `cdrs_annotation.json` mapping.
  - Loads the full complex (Nanobody + Antigen) into AntiFold, freezing the antigen as structural context.
  - Reads the dynamic JSON to mathematically mask ONLY the variable-length CDR residues (bypassing static FASTA misalignment issues).
  - Outputs `AntiFold_Best_Designs.zip`, which packages the top redesigned FASTA sequences alongside the original full-complex PDBs.

## Notebook 4: ESMFold Validation (`4_ESMFold_colab.ipynb`)
- **Objective:** Provide independent, orthogonal validation of the sequence adopting the desired fold.
- **Key Actions:**
  - Uses BioHub's `ESMFold2` API to fold the entire AntiFold-redesigned complex sequence *ab initio* (from scratch).
  - ESMFold operates purely from sequence embeddings and does NOT use the PIPPack coordinates as an initial guess.
  - The resulting structure is then rigidly aligned over the PIPPack reference design to visually calculate the RMSD, verifying if the newly designed CDR sequences naturally adopt the intended structural conformation in reality.

---
**Why this pipeline?**
By replacing RFantibody (which only outputs backbones) with IgGM, and automating PDBFixer into PIPPack, we avoid corrupted coordinate tensors. AntiFold is then fed the full complex with dynamically calculated CDR masks, ensuring the antigen dictates the redesign. Finally, ESMFold blindly folds the sequence as a robust sanity check of the physical fold.
