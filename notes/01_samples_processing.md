I want to really understand where this data came from, specifically in this file, how the samples were processed.

DATASET PAGE on EBI's PRIDE:
https://www.ebi.ac.uk/pride/archive/projects/PXD007647

## Samples
"A number" (I see 34 raw files) of mecillinam resistant mutants from E. coli MG1655

## Sample Processing
> Aliquots containing 50 or 100 μg of each sample were digested with trypsin using the filter-aided sample preparation (FASP) method.

Trypsin digestion is the standard, it cuts after Lysine and Arginine mostly.
FASP is the method with a filter where you wash away the SDS detergents and salts that were used for the cell breaking process, and then you add the trypsin, so broken down peptides can pass through and undigested ones remain on the filter.

> Briefly, protein samples were reduced with 100 mM dithiothreitol (DTT) at 60°C for 30 min, transferred on 10 kDa MWCO Nanosep centrifugal filters, washed with 8M urea solution and alkylated with 10 mM methyl methanethiosulfonate in 50 mM TEAB and 1% sodium deoxycholate.

Ok, this is the description of the FASP protocol it seems.
DTT breaks the disulfide bonds, denaturing the proteins, so trypsin can cleave them more easily.
The MWCO (Molecular Weight Cut Off) filters block anything larger than 10 kDa, **which is basically all proteins** with more than 100 amino acids.
Urea is used to keep proteins in the denatured state.
Alkylation is done to cap the sulfur groups that are exposed after being broken by DTT, otherwise they would just make the sulfide bonds again!
TEAB (triethylammonium bicarbonate) and SDC (sodium deoxychlate) are a pH buffer and a detergent to keep the hydrophobic proteins soluble.

> Digestion was performed in 50 mM TEAB, 1% sodium deoxycholate at 37°C in two stages: the samples were incubated with 500 or 1000 ng (trypsin:protein ratio 1:100) of Pierce MS-grade trypsin (Thermo Scientific) for 3h, then 500 or 1000 ng more of trypsin was added and the digestion was performed overnight.

This is just standard procedure

> The peptides were collected by centrifugation, labelled using TMT 10-plex isobaric mass tagging reagents (Thermo Scientific) according to the manufacturer instructions. The labeled samples were mixed, sodium deoxycholate was removed by acidification with 10% TFA. The combined labeled sample was fractionated using either strong cation exchange chromatography (SCX) or basic reversed-phase chromatography (bRP-LC).

TMT (Tandem Mass Tagging) is a tagging system, in this case 10 labels with the same chemical mass. This is basically like multiplex sequencing in Illumina.
SCX is a fractionation technique that separates peptides based on the positive electrical charge, bRP-LC instead separates based on hydrophovicity. But **they say they separated using "either", so do different samples have different methods?** ***Could there be any batch effect here?***

> quote

In the remaining text they explain the two protocols used for SCX and bRP-LC in more detail.
