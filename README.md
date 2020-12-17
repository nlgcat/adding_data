# Studying the Impact of Filling Information Gaps on the Output Quality of Neural Data-to-Text
Code and other material for the INLG2020 paper [Studying the Impact of Filling Information Gaps on the Output Quality of Neural Data-to-Text](https://www.aclweb.org/anthology/2020.inlg-1.6/).  Also included, are the data files from the experiment in the [SportSett](https://github.com/nlgcat/sport_sett_basketball) paper.  I encourage anyone who wants to use data in this domain to read the SportSett paper first, as it explains the data modelling problem, as well why the training/validation/partition scheme has been changed (there is cross-partition contamination in Rotowire on the encoder end).

## Filling Information Gaps

You will find in the [filling_information_gaps_paper](https://github.com/nlgcat/adding_data/tree/main/filling_information_gaps_paper) folder, our data for the Information Gap paper.

### Datasets (D1 and D2)

D1 is a close representation of the input data in [Clément Rebuffel's](https://github.com/KaijuML/data-to-text-hierarchical) system, except in yearly partition form (see SportSett paper data description below).  We do not use the exact data because there are a couple of slight differences between Rotowire and SportSett.  The most noteable is that we do not have the "Starting Position" for the player in SportSett yet, we had to add some games which were absent from Rotowire, and need to find the data to add.  The database structure of Rotowire does support this, it is just a TBC upload.  D1 contains everything D1 does, plus the additional GAME and NEXT-GAME entities described in the paper.  The important thing when doing thes ablations was not that we use the exact Rotowire input, this varies from system to system anyway, with different author processing their data differently to present to OpenNMT.  Some works will even add extra data without performing abalations, such as adding the day of the week.  Please don't do this, it means people cannot tell if your results are because of the data or your system configuration (unless you re-run the results of prior work with the same data).  Treat the subset of data that is included as a variable!

Please note, there is no JSON for the D1 dataset.  These JSON files were used as input for the RG metric, and we used the same JSON file for both as we need to use the same evaluation for both D1 and D2.  The D2 JSON is inherently a superset of D1, so we use it.

### Abstract Sentences
The [abstract sentence files](https://github.com/nlgcat/adding_data/tree/main/filling_information_gaps_paper/abstract_sentence_count) contain an ordered dictionary in JSON format, where each key is the abstract text, and each value is the number of times it occurs in the respective partition.  For example, the most common abstract sentence in the training partition is:

    "DET-ORG PROPN-GPE PROPN-ORG PUNCT-( X-CARDINAL_COMPARISON PUNCT-) VERB-defeat DET-ORG PROPN-GPE PROPN-ORG PUNCT-( X-CARDINAL_COMPARISON PUNCT-) X-CARDINAL_COMPARISON ADP-on PROPN-DATE PUNCT-.

This would come from a sentence like:

    "The Miami Heat (12-4) defeated the Chicago Bulls (8-8) on Monday."

With 140 such sentences in the corpus, we should make sure that our systems have available all data attributes what might be required to complete the sentence.  Initially we only look at replacing nouns (PROPN and NOUN), but ultimately systems need some way to handle the verbs as well, either allowing the neural model to handle it based on the attributes, or by including something as input to inform the system of when it is true that one team defeated an other.

The code for parsing the raw sentences to this form will be uploaded soon.  It is quite simple though, just using a couple of custom rules in [spaCy](https://spacy.io/) to macth things like team records in parenthesis, or player shot breakdowns like "(4-6FG, 0-4FT)".  We hope to also improve this functionality to group similar sentences together, as you can see in the files the most common sentence types are quite similar, often describing the initial game.

## Datasets from the SportSett paper

In the [sport_sett_paper](https://github.com/nlgcat/adding_data/tree/main/sport_sett_paper) you will find the data for the experiment in the original SportSett paper.  Here we started with the original partition scheme of Rotowire (contaminated), then, we fix the contamination issues by moving game records such that each game only appears in one partition (decontaminated).  Finally, because we believe that with the original partition schemes, models can learn things which they should not have sight of (such as which dates map to which days of the week), we introduce the yearly partition scheme, where we train on 2014-16, validation 2017 and test on 2018.  We have received several comments on this.  Firstly, we were asked if we should test different year combinations, as well as how many years are in each parition.  The short answer is ideally yes.  The long answer is the ablation experiments took 2-3 weeks on a pair of 2080ti's.  We have also seen several comments such as "but how can the model learn about a season, or the dates, or the teams, if it does not have access to the data?".  I believe this is funtamentally the wrong way to look at the problem.  If a sports media company wants you to create an NLG system like this, to be deployed in 2022 (I am writing this in 2020), they will give you the previous data and texts, but you can hardly ask them for the 2022 game data!  Systems need to learn on previous data, then correctly simulate human behavious on future data.  Slicing historic data randomly, then considering the withheld data as "future" is a form of contamination.

## Our work and shared task on evaluating factual accuracy in this domain

Please also see our work on [evaluating accuracy](https://github.com/nlgcat/evaluating_accuracy) in these systems, which often generate about 20 factual errors per summary!  We are also proposing a [shared task](https://github.com/ehudreiter/accuracySharedTask) which aims to find better metrics for this domain (and dataset in particilar), using our evaluation protocol as a gold standard with which to correlate cheaper methods/metrics.


