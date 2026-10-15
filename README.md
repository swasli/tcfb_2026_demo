# tcfb_2026_demo

My homework repo for TFCB 2026. Made by Gadis Shahjehan (@swasli).

## Folder layout

- `experiments/` : experiment folders, including the tidy qPCR table
- `coldspots_coordinates.csv` : coldspot coordinates with IDR calls
- `coldspots_analysis_report.pdf` : coldspot analysis report
- `alphafold x coldspots (1).zip` : ChimeraX images for the sampled genes
- `README.md` : this file
# Visualizing Coldspots in Genes using ChimeraX

Do coldspot regions in genes code for intrinsically disordered regions (IDRs) of proteins? This repo walks through a simple way to check, using AlphaFold structures in ChimeraX.

## Table of Contents

- [Introduction](#introduction)
- [Hypothesis](#hypothesis)
- [Questions](#questions)
- [What You Need](#what-you-need)
- [Step by Step Methodology](#step-by-step-methodology)
- [Results](#results)

---

## Introduction

Intrinsically disordered regions (IDRs) don't fold into a fixed 3D shape and make up about 25% of the human proteome. Pathogenic variants avoid IDRs, while ClinVar benign variants are enriched there. Coldspots show the same pattern, even stronger. The likely reason is that one amino acid change rarely disrupts a flexible region, but it can break a folded domain.

We want to see if coldspots code for IDRs, using ChimeraX to look at them directly.

## Hypothesis

> **Coldspot regions overlap with intrinsically disordered regions (IDRs).**

**IDR** = a region that doesn't fold into a fixed shape, sits outside a folded domain ("spaghetti straps"), and has low AlphaFold confidence scores (🟠 orange, below 50).

### What is an AlphaFold score?

AlphaFold scores use a metric called **pLDDT** (predicted Local Distance Difference Test) to measure how confident the AI is about its protein structure prediction. AlphaFold colors each part of the protein like a heat map to show this confidence level:

| Color | pLDDT score | What it means |
| --- | --- | --- |
| 🔵 Dark blue | above 90 | Very high confidence. The AI is highly certain this part is correct, so you can rely on these areas for detailed atomic shapes. |
| 🔷 Light blue | 70 to 90 | Confident. The AI is mostly sure about the shape and backbone of this region. |
| 🟡 Yellow | 50 to 70 | Low confidence. The AI is unsure, so treat these shapes with caution. |
| 🟠 Orange | below 50 | Very low confidence. These regions are often unstructured, floppy, or flexible loops in real life (intrinsically disordered). |

## Questions

1. How much of each coldspot falls on a disordered region?
2. Do coldspots land on disordered regions more often than we'd expect by chance?
3. Is the pattern consistent across genes, or driven by a few?

## What You Need

| What | Why |
| --- | --- |
| [regional_coldspots.tsv](https://github.com/user-attachments/files/33451446/regional_coldspots.tsv) | The coldspot data. You only need the `chr` and `amino_acids` columns. |
| [ChimeraX](https://www.cgl.ucsf.edu/chimerax/download.html) | To view the protein structures. Install the version for your device, and watch the [tutorials](https://www.youtube.com/watch?v=8XmbMROKNnQ&list=PL4eF1KHNgDfIYSKCS3_S0PTRYtYTV9Myi) to get used to the interface! |
| Microsoft Excel or Google Sheets | To record your genes and results. |
| Python (optional) | To pick random genes. |

## Step by Step Methodology

### Step 1: Pick random genes

Open the [regional_coldspots.tsv](https://github.com/user-attachments/files/33451446/regional_coldspots.tsv) file. You only need the `chr` and `amino_acids` columns. Pick around 30 genes (`chr`) at random. You can use the Python code below, or any method you prefer.

```python
import pandas as pd

df = pd.read_csv("regional_coldspots.tsv", sep="\t")

GENE_COL = "gene"   # change this to your gene symbol column
SEED = 42

# keep one row per gene so no gene gets picked twice
genes = df.drop_duplicates(subset=GENE_COL)[[GENE_COL, "chr"]]

pick = genes.sample(n=30, random_state=SEED)
print(pick)
print(pick["chr"].value_counts())   # quick look at how they spread across chromosomes

pick.to_csv("random_30_genes.tsv", sep="\t", index=False)
```

This prints out random genes from the thousands in the file.

> [!IMPORTANT]
> Some genes (`chr`) have more than one coordinate for a coldspot. You can keep all of them for the same gene, or pick just one coordinate per gene. Keeping multiple coordinates increases your sample size and power, but keep this in mind when you analyze and calculate the averages at the end of the experiment.

### Step 2: Set up your spreadsheet

Make a separate Excel or Sheets file and label it **`coldspots_coordinates`**. Record the random genes you chose, using only the `chr` and `amino_acids` columns from the regional_coldspots file. Then add two more columns:

- **`uniprot_id`**: where you put the protein IDs (shown how in Step 3)
- **`idr`**: where you write YES or NO for whether the coldspot falls on an IDR
  - **YES** = falls in the IDR and outside the main domain
  - **NO** = falls within the folded domain

### Step 3: Find the UniProt IDs

Go to the [UniProt ID Mapping](https://www.uniprot.org/help/id_mapping) website and put in the gene name (`chr`). In the search results, look for the **Entry** column. This ID is what ChimeraX uses to know which protein you want to visualize. Make sure you pick the **human** protein and not another species!

**Example:** for the gene SLU7, ID Mapping shows the entry as [O95391](https://www.uniprot.org/uniprotkb/O95391/entry), labelled `SLU7_HUMAN`. The ID `O95391` is the one you want.

### Step 4: Double check

This is just to make sure you're on the right path!

1. Sit back, take a breather, stretch, walk around for a bit, and come back!
2. Check that your Excel or Sheets file has these 4 columns:

| Column | What goes in it |
| --- | --- |
| `chr` | name of the genes you randomly picked |
| `amino_acids` | coordinates of the coldspot |
| `uniprot_id` | protein IDs from the UniProt ID Mapping website |
| `idr` | YES / NO, filled in after you look at the structures in ChimeraX |

3. Give yourself a pat on the back for doing so well!

### Step 5: Visualize in ChimeraX

By the end of this step, you will have:

- **one** AlphaFold labeled image per gene
- **one or more** coldspot labeled images per gene (depending on how many coordinates you kept for each gene)

#### 5a. AlphaFold confidence image

You need the `uniprot_id` and `amino_acids` for each gene from your file. Type these commands into the ChimeraX command line, replacing `O95391` with your UniProt ID:

```
alphafold fetch O95391
hide atoms
show cartoons
color bfactor palette alphafold key true
view
```

Save the image and label it **`genename_alphafold`**, for example:

```
save SLU7_alphafold.png width 1600 supersample 3
```

#### 5b. Coldspot image

Now let's label where the coldspots are! Enter these commands in the same window, replacing `100-250` with your coldspot's amino acid start and end. You can pick any color you want for the coldspot region.

```
color #1 lightgray
color #1:100-250 cornflowerblue
```

Save the image as **`genename_coldspot`**, or **`genename_coldspot1`**, **`genename_coldspot2`** and so on if you have multiple coordinates.

```
save SLU7_coldspot1.png width 1600 supersample 3
```

> [!TIP]
> If it's hard to see where the coldspots are, rotate the 3D model or zoom in to find a better angle. This is up to you!

> [!NOTE]
> The numbers after `#1:` must be **amino acid positions**, not genomic coordinates. If you type genomic positions, ChimeraX won't find anything to color.

Do this for every coordinate in your file! Then save and compress all the images into one folder called **`coldspots_images`**.

### Step 6: Score each coldspot

Now you have images of every coordinate showing the AlphaFold scores and the coldspot regions, so compare them!

| What you see | What to write in `idr` |
| --- | --- |
| Coldspot falls on the 🟠 orange or 🟡 yellow region | **YES** |
| Coldspot doesn't fall on those regions | **NO** |
| You're unsure, or there's no data | **N/A** |

Save and export the file as **`coldspots_coordinates.csv`**.

### Step 7: Analyze

This is where we analyze quantitatively. Using the total number of coordinates you picked (30 or more), calculate:

- **YES / total**
- **NO / total**
- **N/A / total**

## Results

By the end, you should have:

| Output | File name |
| --- | --- |
| Protein visuals of the coldspot regions and their AlphaFold scores | `coldspots_images` (compressed folder) |
| Spreadsheet with the YES / NO / N/A IDR calls, plus the columns from Step 2 | `coldspots_coordinates.csv` |
| A short report with the analysis from Step 7 (feel free to use better methods!) | `coldspots_analysis_report` (PDF or Word) |

### I invited my TA and a classmate as collaborators. Before the invite they could only view the repo, but now they have write access so they can push commits, edit files, and be assigned to issues. They also get notified about activity in the repo.


## Homework 1
- [Issue 1](https://github.com/swasli/tfcb_2026_demo/issues/1)
- [Issue 2](https://github.com/swasli/tfcb_2026_demo/issues/2) 
- [Log File](https://github.com/swasli/tfcb_2026_demo/tree/main/experiments)
- [Commit](https://github.com/swasli/tfcb_2026_demo/commit/334765a0cbb8cd01580a8ab1de89bf285d810780)
- [Labels](https://github.com/swasli/tfcb_2026_demo/labels)
