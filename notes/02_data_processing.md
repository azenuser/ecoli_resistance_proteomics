## The mass spectrometry experiment pipeline
- We start with 40 E. coli samples (20 resistant, 20 controls)
- Inside of each E. coli organism there are roughly 4000 protein types.
- Trypsin digests those proteins into roughly 100'000 to 200'000 unique peptide fragments.
- we use a Liquid Chromatography column connected to the mass spectrometer to separate the proteins that actually enter the machine (by hydrophobicity, ph or by electrical charge) through time.
- MS1, the first scan of the machine is like a snapshot of what just exited from the liquid chromatography column, there could be 10 peptides lets say
- targeting, the machine picks one of the peaks of those peptides, selecting and collecting only that for the second scan
- MS2, the second scan of the machine, where it records the raw spectrum from the broken fragments of that one specific peptide selected in the targeting step.
- Now that the LC-MS/MS procedure is done, we would have something like 50'000 MS2 spectra from this one run.

Lets take spectra number \#12 and see how it is processed:
- The search engine (Mascot) takes the fragment pattern of this spectra and compares it against a DB made of the combination of a **real, target database** and a **fake, decoy database**.
- The algorithm reports the best match with its score, it could say that out of all the possible peptides in the reference database (target + decoy), the best match is given by the target peptide `VGAHAGEYGAEALER` with a certain score.
This is, of course, done for each spectra (of which we have one per peptide fragment)

At the end of the search we have a table with our 50'000 rows, one per spectra, where the columns may be something like:
spectrum ID,   Best match found in database,   Database type,   Score
spectrum \#12   `VGAHAGEYGAEALER`              **Target**       94.7
spectrum \#98   `LGEAAEGAYEGAHAV`              **Decoy**        12.6

To filter out the random noise, the software ranks all of the 50'000 scores from highest to lowest, and then it starts keeping count of the number of false positives (the ones that matched the decoy database), and if they go over a set threshold (in this work 1%), it stops, throws away all the remaining rows, while the rows above assign the spectrum to the best match peptide from the database.

### It feels like a waste?
To me it felt like a waste that so many lines could be discarded, but thinking about it, the peptides still come out dilationed from the liquid chromatography column, so it could be that the signal for the same peptide comes out multiple times, and one of the times it will be read perfectly, while other times it may result in a poor S-to-N ratio, getting a very bad score.
Also, the MS/MS machine is always working, so its possible that it also reads "waste" coming out of the column, without actual peptides, so we need to discard those, and they will probably have a very low score, as they dont match the peaks present in real biological databases.

## Interesting ideas
**Making the MS machine more efficient**
It could be useful if a MS machine could have a "timer" for a certain mass/peak, so that after it did an MS2 run on a specific peak, it wouldnt try it again after one second on that same peak, as it could simply be the same peptide coming out of the column a bit later. - **AH THIS EXISTS APPARENTLY, ITS CALLED "DYNAMIC EXCLUSION"**
**Understanding TMT tags**
The idea of the isobaric tags is that they are the same weight, so they all exit the column at the same time with the peptides (TMT tags attach to Lysine and N-terminus as they are exposed by trypsin). And we use one for each biological sample. So during MS1 we get a greater peak thanks to the same mass of the TMTs, which gets selected, and the tags break off during the MS2 scan. 
So in the raw spectra fromthe MS2 we see the spectra of the peptide, PLUS the "counting" of the tags, one size per sample (we have 10-plex so 10 different sizes) and with this we can evaluate the presence of the peptide among the different samples. So it really is like the multiplexing in Illumina!


## Data Processing

> Data analysis was performed using Proteome Discoverer version 1.4 or version 2.1 (Thermo Fisher Scientific).

Ok, Proteome Discoverer is Thermo Fisher proprietary software for the processing of MS data. In theory the pipeline it follows is:
- Spectrum Preprocessing, gets list of peaks from the raw spectra
- DB searching, matching the experimental spectra against a protrein database (algorithms like Sequest and Mascot), scoring the matches and inferring which peptides and then proteins were in the sample.
- Quantification, reads the TMT reporter ion intensities (from the 10-plex tags) for each identified peptide.
- Protein Assembly / Rollup, aggregation of the identified peptides and reporter ions intensities up to the protein level.

> [!NOTE]
> They say they used **version 1.4** or **version 2.1**, are the outputs different? Will there be any kind of ***batch effect due to changes in the scoring algorithms*** or something like that?

> The SCX data was processed with Proteome Discoverer 1.4 using the Swissprot database (November 2014) with Escherichia coli taxonomy filter. Mascot 2.3.2 (Matrix Science) was used as a search engine with precursor mass tolerance of 5 ppm and fragment mass tolerance of 0.2 Da. No missed cleavages were accepted, mono-oxidation on methionine was set as a variable modification, methylthiolation on cysteine and TMT-6 reagent modification on lysine and peptide N-terminus were set as a fixed modification.

Ok, for the SCX data they used version 1.4, they used swissprot, which is the manually curated version of the protein data bank, filtered specifically over E. coli proteins.
They used the Mascot search engine algorithm.
"No missed cleavages were accepted" means that if there were any peptides with an internal lysine or arginine, those **were discarded**. It is a missed site because trypsin should have cut there!

Now, the part about variable and fixed modifications:
- fixed modifications are things you tell the algorithms will always be found in the modified state. You must have somehow altered these aminoacids to always present a certain modification. For example you can find in the 01_sample_preparation.md document that they used MMTS to cap (by methylthiolation) the -SH free ends of all cysteines (to -S-S-CH3), so all those aminoacids will have an altered m/z. This is also true for the N-term ends of all lysines which have the molecular tags added.
- variable modifications are the same thing, but you tell the search engine to check both a version with the modification and one without.
these changes in m/z are used so that the algorithm (Search Engine) modifies the mass representations of each candidate peptide (the ones that get built by breaking down the proteins from swissprot), in this way they more closely match what we see in our raw spectra.

But **why is methionine oxidation set as variable?** Because the S groups of methionine can react with the oxygen in water during sample preparation, but this is spontaneous and incomplete, so we need to find both possibilities.

> The target-decoy approach was used for the filtering of the identifications, with the target false discovery rate of 1% as a threshold to filter confident peptide IDs.

The target-decoy approach is used to estimate and control how many of the identified peptides are false positives (as found by the search engine algorithm). You have a **target database** (the real organism proteome, all the E. coli protein sequences), and a **decoy databse** (a randomized version of the real sequences, or the reverse of every aa sequence, which becomes nonsense). Then when you run the search, you do so against the combination of those two databases, this gives you a false discovery rate, because every match in the decoy DB is wrong by definition.
