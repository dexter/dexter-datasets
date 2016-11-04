## SEL: A Unified Algorithm for Entity Linking and Saliency Detection



This directory contains the dataset used for writing the paper 
"SEL: A Unified Algorithm for Entity Linking and Saliency Detection", 
if use this data for your experiments, please cite our paper: 

	@inproceedings{trani2016sel,
	      title={SEL: A Unified Algorithm for Entity Linking and Saliency Detection},
	      author={Trani, Salvatore and Ceccarelli, Diego and Lucchese, Claudio and Orlando, Salvatore and Perego, Raffaele},
	      booktitle={Proceedings of the 2016 ACM Symposium on Document Engineering},
	      pages={85--94},
	      year={2016},
	      organization={ACM}
	}

### How the dataset has been generated

Starting from an English dump of Wikinews, we sampled 400 WikiNews articles published from November 2004 to June 2014 and having between 10 and 25 linked entities per news. With the help of 4 expert annotators and using the [Elianto](https://github.com/dexter/elianto) annotation framework, we identified the entity saliency score in a specific subset of 62 of these documents. We then exploited CrowdFlower for annotating linked entities with saliency scores. Each document has been annotated by at least 3 CrowdFlower users. The golden dataset produced by the expert annotators has been used as a quality control mechanism, in order to detect and ban malicious annotators. The resulting dataset has then been analyzed and several documents were discarded due to a low agreement between the annotators. The final dataset is composed by 365 documents, with 12.02 entities per document on average.

### Contents: 

The dataset is composed of two files: a JSON file, describing the original dataset, and an SVM file, describing the features extracted for each entity-document pair.

#### JSON file

The `saliency-dataset.json` file contains the raw dataset produced using the CrowdFlower platform and the Wikinews articles selected as described above. Each row in this file contains a separate json representation of a single news, with the following fields:

- docId: the internal ID of the document inside the dataset
- title: the title of the news, taken from Wikinews
- timestamp: the date and time the news appeared on WikiNews
- wikiTitle: the name of the page on the Wikinews portal (e.g., the first document is titled 
`Iran_close_to_decision_on_nuclear_program` and is uniquely identified by the page https://en.wikinews.org/wiki/Iran_close_to_decision_on_nuclear_program]
- wikinewsId: the ID of the news on the Wikinews portal
- document: the document content, described by a list of (name, value) tuples, where the names identify the name of the field and the value its content. Every news in this dataset is described by the following fields:
	- headline: the headline of the news (i.e., its title)
	- dateline: the date of the described event
	- body_par_*: a list of fields, with the content of the body paragraphs, with their relative order (i.e., 000 precede 001 and so on)
- saliency: the saliency scores of the entities annotated in the news by the Wikinews editors. This field is described with a list of tuples composed by the following fields:
	- entityid: the ID of the Wikipedia entity. Resolving IDs can be done using the Wikipedia URL: https://en.wikipedia.org/w/index.php?curid=9282173
	- score: the saliency score of the entity, as recognized using the CrowdFlowers users.

#### SVM file

The `saliency.svm.gz` file, on the other hand, is the dataset compressed with gzip and described in the SVM-light format. This file has been used for splitting the dataset in training/validation/testing, and then for training our models and testing their effectiveness. 

Each row in the SVM file describes a single entity referring to a specific document. 
The document is identified by the ID described in the second column, while the comment of the row (i.e., everything which follows the # character at the end of each row) include some additional information regarding the specific entity. 
For instance, the first row is the following:

	-1.0 qid:1 1:0.60355 ....  138:3.510496 #     [powell]        9350062 Arthur_William_Baden_Powell

This row identify the entity with ID = 9350062 and Wikipedia-title = Arthur_William_Baden_Powell related to the spot `powell` in the document with ID = 1. Its saliency score is -1, which means it is an entity not relevant for the document (these non-relevant entities have been produced by a spotting process executed on the same spots of the linked entities, in order to produce negative example useful for training our models).

The intermediate columns identified by a [index]:[value] representation, describe the values of the features extracted for each entity-document pair. The first 39 features (i.e., from index = 1 to index = 39) describe the light features, as described in the articles. These feature are fast to compute. The features with index = 40 and 41 are the tagme-like voting schema features, while the feature with index = 42 is the confidence score of the binary classifier used in the first step of our two-steps approach. The remaining features (from index = 43 to index = 138) describe the graph features, that together with the tagme-like features compose the set of heavy features, i.e., expensive to compute.