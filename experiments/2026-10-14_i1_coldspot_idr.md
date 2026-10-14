# Check if coldspot regions overlap with disordered protein regions using ChimeraX

2026-10-14

Issue: [#1]([https://github.com/swasli/tfcb_2026_demo/issues/1](https://github.com/swasli/tcfb_2026_demo/issues/1#issue-5856012716))

## Data and tools

| file or tool | source | what I used it for | how much I used |
| --- | --- | --- | --- |
| regional_coldspots.tsv | course file | list of genes and coldspot positions | 20 random genes |
| UniProt ID Mapping | uniprot.org | finding the protein ID for each gene | 1 ID per gene |
| AlphaFold structures | AlphaFold Protein Structure Database | 3D protein models with confidence scores (pLDDT) | 1 model per gene |
| ChimeraX | UCSF | viewing the proteins and highlighting the coldspots | 2 images per coldspot |
| Python (pandas) | python.org | picking the random genes | seed 42 |

## Steps

1. Picked 20 random genes from `regional_coldspots.tsv` using Python.
2. Found the human UniProt ID for each gene and added it to `coldspots_coordinates.csv`.
3. Opened each protein in ChimeraX with `alphafold fetch <uniprot_id>`.
4. Colored the protein by AlphaFold confidence and saved the image.
5. Colored the protein gray, colored the coldspot (aa_start to aa_end) blue, and saved the image.
6. Compared the two images and wrote YES if the coldspot was on an orange or yellow (floppy) region, NO if it was on a blue (folded) region, and N/A if there was no model.
7. Counted the YES, NO, and N/A results.

## Result

![AlphaFold pLDDT and coldspot views of MAST3 and FGD1](img/mast3_fgd1_coldspots.png)

*MAST3 and FGD1 colored by AlphaFold confidence (left) and with the coldspot highlighted in blue (right). In both proteins the coldspot sits on the floppy, low confidence loops outside the folded core. Image made by Gadis in ChimeraX using AlphaFold models.*

| result | number of coldspots |
| --- | --- |
| YES | 28 |
| NO | 1 |
| N/A | 30 |
| **Total** | **59** |

28 out of 29 coldspots I could score landed on disordered regions. Most of the N/A ones come from KMT2B and ALMS1, which are too long for one AlphaFold model.
