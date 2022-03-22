# Proten inference algrithm 

This is probably the first realization of maxquant protein inference algrithm, in R as well

Protein inference is technically a mission impossible, because of the information lost during the digestion. 

There are a lot of tries, some all very simple, just list all possible proteins, or very complicated, with some probability based method to predict the most likely existing proteins. 

No matter how complicated the algorithm works, it basically can not tell a shared peptide's origin.  

Reporting the protein list is just one thing, the quantification is another more important thing. The protein inference matters a lot in how to organize peptide intensities to protein intensities. For example, FlashLFQ does a very good job in peptide quantification, but not in protein inference and therefore in protein quantication. Several datasets tested in-house showed very weired result on protein level, but normal(as expected) on peptide level.

Maxquant actually uses a very simple protein inference way, which can be concluded into two points:
* winner takes all (occama's razor)
* shared peptides only quantified once

It is a compromise between "not using shared peptide" (MSstatTMT) and using all shared peptide (which is anothe option in FlashLFQ used), and makes sure that all peptides (quantification) information are used in the protein list, but not over-used (in FlashLFQ, shared peptide are used for all proteins mapped) 
An easy common sense here is that the expression profile reprented by peptide and protein should be the same (or more accurately saying, similar). So far, we found the razor peptide way is one of the best to presever the quantifictaion profile from peptide to proteins.  

Many users complain that Maxquant does not open source their protein inference algorithm. Here I tried to realize it in R. From the code presented here, I only used peptide.txt as input, (precisely I only used 2 columns, peptide id and protein id) can get very similar result to the proteinGroups.txt.
The algorithm only need the peptide-protein mapping file (gc format, which is peptide_id protein1;protein2,protein3), does not need fasta at all. 

Feel free to test it. 


# Logic

* Take the peptide-protein mapping as input
* Reverse the mapping, to have a protein-peptide mapping
* Order the table according the number of peptides mapped to each protein
* Start from the top1 protein (leading protein), iterate the whole table from the 2nd one, find all sub proteins (with peptide ids as subset of the first protein), group them all together as a protein group an store it in another table. Then delete all these proteins from the table
* Do the above step, until there is not table is empty 


# Some questions

##  How to deterimin a razor peptide?

In the newly generated protein group table, as long as a peptide occurs first, it is a razor peptide. This is true because of the way we generate the proteingroup table. A peptide could be assinged to multiple protein groups, but if it is assgined before, it will be not a razor peptide, and will not be used for quantification.

## how to determin a unique peptide?

One thing needs to note that unique peptide in Maxquant, means proteingroup unique, instead of protein unqiue. Here in this project, protein unique is called distinct peptide, which is also listed as one seperate column. 
The protein group uniqueness is determined afterwards, when the whole protein group table is finished. The check if peptide is only assgined to one protein group, it is then a unquie peptide. 

## Some uncertainty. 
As you can see that there will be some uncertainty. When ordered by peptide counts, a razor peptide might be assigned to different proteingroups just because a protein is listed above another. Yes, this might be the reason why here the result is slightly different from Maxquant result (Possible some other reasons). check the example included:

* peptide.txt: direct output from maxquant, used as input here
* proteinGroups.txt: directly ouput from maxqunat
* proteinGroups_homebrew.txt: output here. 










