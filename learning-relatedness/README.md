## Learning Relatedness Measures for Entity Linking Dataset

This directory contains the dataset used for writing the paper "Learning Relatedness Measures for Entity Linking Dataset", 
if use this data for your experiments, please cite our paper: 

```

@inproceedings{ceccarelli2013learning,
        title={Learning relatedness measures for entity linking},
        author={Ceccarelli, Diego and Lucchese, Claudio and Orlando, Salvatore and Perego, Raffaele and Trani, Salvatore},
        booktitle={Proceedings of the 22nd ACM international conference on Information \& Knowledge Management},
        pages={139--148},
        year={2013},
        organization={ACM}
}
```

### Code used for generating the dataset 

you can check out the code used for the experiments by running: 

```
git clone http://git.hpc.isti.cnr.it/ntt/dexter-relatedness.git
```

### Contents: 

`relatedness-assessment.tsv.gz` contains the couples of mentions (spots)
that we extracted from the documents together with the entities, each line
is formated in this way:

`<id doc> <e spot> <id e> < candidate spot > <id c> <relatedness [1|0]>`

Where `e` is the known entity while `c` is the candidate entity (the int represents the wikipedia id of the entity)

If you need to generate the features have a look to the class FeatureGenerator.
You would need to patch the function:
    
```java
public List<Double> eval(int e, int c) {
        helper.addEdges(inE, outE, inC, outC);
```
and replace `inE` entities that link to the entity `e` and `outE` with the entities
linked by the entity `E`.. the same for `C`.

Finally, `test.svm.gz`, `validate.svm.gz`, `training.svm.gz`
are the datasets that we created to learn the model.
Each file contains a distinct subset of the documents in conll, and
for each subset the features of the couple of spots found in the document.
(the comment at the end of each line contains the ids of the entities followed
by the conll document id from where we extracted the couple)
You can see the order of the features in the method
`FeatureGenerator.getStdFeatureGenerator`

### FAQ

> Could you please tell me how did you compute `P@K`? Did you need a validation
> set to find a optimal threshold to judge semantically-related not first?

We generated the dataset from the conll
dataset, considering one mention at a time, together with its
correctly associated
entity, and then generating the set of 'semantically related' entities
as the correctly associated entities of the near mentions (where
'near' means co-occurring in a 50 terms window). while the set of
semantically-unrelated entities was produced in the same way, but
using the not-correct disambiguation of the near mentions.

As example, given a document manually annotated:

```
On this day 24 years ago Maradona scored his infamous "Hand of God"
goal against England in the quarter-final of the 1986 
```

where Maradona is correctly linked to the football player and England
is the Football Team (and not the country, nor  the rugby union team)
I would generate for the entity Maradona the following golden truth:

``` 
Maradona - England(football team) - relavant
Maradona - Single-elimination_tournament- relavant (the quarter-final mention)
Maradona - English rugby union team  - not relavant
Maradona - England(country) not relevant
[...]
```

> When you computing `P@1,5,10`, did you just use 
> `P@k = (number of relevant entities in top ranked k positions)/K?`
> If you used this, if the number of relevant entities is 
> less than k, then it tends to make the scores lower. 
> I just want to make sure I got your evaluation method 
> clearly since I didnot see exactly what are the formulas you used exactly in the paper. 

You are right, but for how precision is defined, you relevant entities
are less than `k`, you
still divide per `k`, to avoid what you observed. If you want to be sure
that you are computing
the measures like me, we used the latest version of the standard
`trec_eval` library ([http://trec.nist.gov/trec_eval/trec_eval_latest.tar.gz]) 

> Could you please tell me which dataset you used for the entity linking 
> experiments (scores reported in Table 4)? Is it a CONLL or
> AIDA dataset? And did you use any dataset from CONLL testb 
> in the training set you used to learn the learning to semantic relatedness measure?

It was the AIDA dataset (built from Conll), We used the whole dataset
(so the training + testb)
and we partitioned the documents in a training, a validation and a
test set. Results reported was
on the test-set, but I don't think that it was test b. If you need I
could investigate which documents
were in the testset.
