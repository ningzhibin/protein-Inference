# Proten inference algrithm 

This is probably the first realization of maxquant protein inference algrithm, in R as well

Protein inference is technically a mission impossible, because of the information lost during the digestion. 

There are a lot of tries, some all very simple, just list all possible proteins, or very complicated, with some probability based method to predict the most likely existing proteins. 

No matter how complicated the algorithm works, it basically can not tell a shared peptide's origin.  

Reporting the protein list is just one thing, the quantification is another more important thing. The protein inference matters a lot in how to organize peptide intensities to protein intensities. For example, FlashLFQ does a very good job in peptide quantification, but not in protein inference and therefore in protein quantication. Several datasets tested in-house showed very weired result on protein level, but normal(as expected) on peptide level.

Maxquant actually uses a very simple protein inference way, which can be concluded into two points:
* winner takes all (occama's razor)
* shared peptides only quantified once

It is a compromise between "not using shared peptide" (MSstatTMT) and using all shared peptide (which is anothe option in FlashLFQ), and makes sure that all peptides (quantification) information are used in the protein list, but not over-used (in FlashLFQ, shared peptide are used for all proteins mapped) 
An easy common sense here is that the expression profile reprented by peptide and protein should be the same (or more accurately, similar). So far, we found the razor peptide way is one of the best to preseve the quantifictaion profile from peptide to proteins.  

Many users complain that Maxquant does not open source their protein inference algorithm. Here I tried to realize it in R. From the code presented here, I only used peptide.txt as input, (precisely only 2 columns, peptide id and protein id) can get very similar result to the proteinGroups.txt.
The algorithm only need the peptide-protein mapping file (gc format, which is peptide_id protein1;protein2,protein3), does not need fasta at all. 

Feel free to test it. 

R code in protein inference.Rmd


# Logic

* Take the peptide-protein mapping as input
* Reverse the mapping, to have a protein-peptide mapping
* Order the table according the number of peptides mapped to each protein
* Start from the top1 protein (leading protein), iterate the whole table from the 2nd one, find all sub proteins (with peptide ids as subset of the first protein), group them all together as a protein group and store it in another table. Then delete all these proteins from the table
* Iterate the above step, until the table is empty 
* re-examine the list, to tell if the peptides are group unique
* remove protein groups with 0 razor proteins 


# Some questions

##  How to deterimine a razor peptide?

In the newly generated protein group table, as long as a peptide occurs first, it is a razor peptide. This is true because of the way we generate the proteingroup table. A peptide could be assinged to multiple protein groups, but if it is assgined before, it will be not a razor peptide, and will not be used for quantification.

## how to determine a unique peptide?

One thing needs to note is that unique peptide in Maxquant, means proteingroup-unique, instead of protein-unqiue. Here in this project, protein unique is called distinct, which is also listed as one seperate column. 

The protein group uniqueness is determined afterwards, when the whole protein group table is finalized. Then check if a peptide is only assgined to one protein group, it yes is then a unquie peptide. 

## Some uncertainty. 
As you can see that there will be some uncertainty. When ordered by peptide counts, a razor peptide might be assigned to different proteingroups just because a protein is listed above another. Yes, this might be the reason why here the result is slightly different from Maxquant result (Possible some other reasons). check the example included:

* peptide.txt: direct output from maxquant, used as input here
* proteinGroups.txt: directly ouput from maxqunat, for comparison
* proteinGroups_homebrew.txt: output by my script. 

# proteingruops without unique peptides
This algorithm does not try to come out with the shortest protein group list. It only makes sure that each protein group has at least 1 razor peptide, not even 1 unique peptides. Therefore it is easy and works for quantification. 
You will find some proteingroups without any group-unique peptides, some times these proteins even have a very good number of razor peptides. These proteins could be technically removed without any issue, with covering all peptides identified. If you like you can do another round of calculation, by deleting all these proteins with no unique peptides, then re-identify if a peptide is a razor peptides. 

Some further reduction could also be performed, for cases that a subset peptides from a proteingroup can be explained by some other (more than 1) proteingroups. If true, this protein could also be removed. Remember that which protein to keep for this case also does not have one only answer. 

## Exmample: which proteingroup is more like to exist???????

* proteingroup1: peptide2, peptide3
* proteingroup2: peptide1, peptide3
* proteingroup3: peptide1, peptide2

In the algrithm here, 3 protein groups are listed in the primary sorting, then depending on the order, the third one (not necessarily proteingroup3) will be omitted for lacking razor peptides, because all two peptides are alreay used for quantification. 

However, this is only for the purpose of being parsimonious. Personally, for quantification, the concept of razor peptide (shared peptide only qunatified once) is simple and efficient. 









