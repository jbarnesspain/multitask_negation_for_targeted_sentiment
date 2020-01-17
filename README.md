# Multitask incorporation of negation for targeted sentiment classification


The idea is to use multitask learning to incorporate negation information into a targeted sentiment classifier

## Model
We could start with the BiLSTM + CRF model I used for sentence-level classification, but adapted to targeted sentiment. Other models could be cool too though.

The model currently is a BiLSTM + CRF model where BiLSTM has a 50 dimension hidden state.

## Resources
We assume any resource mentioned here is stored within `./resources`.
### Word Embeddings
All embeddings mentioned here have to be downloaded into their respective directories as they are too large for the repository.
#### English
Here we used the [300D 840B token GloVe embedding](https://nlp.stanford.edu/projects/glove/) of which we assume this can be found at the following path `./resources/embeddings/en/glove.840B.300d.txt`.

### Model Configurations
All the model configurations can be found within `./resources/model_configs`

## Datasets
# Sentiment
1. SemEval 2016 Aspect-based datasets (EN, ES)

The sentiment datasets need to be downloaded and converted from BIOSE to BIOUL format. The format change is only label differences e.g. S==U and L==E. The sentiment datasets are from [Li et al. 2019](https://www.aaai.org/ojs/index.php/AAAI/article/view/4643) and they are the SemEval 2014 Laptop dataset and the SemEval 2014, 2015, and 2016 combined Restaurant dataset. The downloaded datasets will go into the following folders `./data/main_task/en/laptop` and `./data/main_task/en/restaurant`. To download run the following script:
``` bash
python targeted_sentiment_downloader_converter.py
```

# Negation
1. EN - [ConanDoyleNeg](https://www.aclweb.org/anthology/S12-1035.pdf), [SFU Review Corpus](https://www.aclweb.org/anthology/L12-1298/)
2. ES - SFU Spanish

For both the ConanDoyleNeg and SFU review corpus the negation scopes and cues are discontinous such that the BIO scheme does not continue if a cue is within a scope or vice versa, an example of this is shown below where the dis-continus version is example 1 (which is used in this project) and the continous version is shown in example 2.

#### Example 1
``` python
'''
Mooner	O	O
simply	O	O
forgot	O	O
to	O	O
show	O	O
up	O	O
for	O	O
his	O	O
court	O	O
appointment	O	O
,	O	O
as	O	O
if	O	B_speccue
he	O	B_spec
could	O	B_speccue
ever	O	B_spec
remember	O	I_spec
,	O	O
and	O	O
'''
```

#### Example 2
``` python
'''
Mooner	O	O
simply	O	O
forgot	O	O
to	O	O
show	O	O
up	O	O
for	O	O
his	O	O
court	O	O
appointment	O	O
,	O	O
as	O	O
if	O	B_speccue
he	O	I_spec
could	O	I_speccue
ever	O	I_spec
remember	O	I_spec
,	O	O
and	O	O
'''
```

## SFU Dataset Splitting
To create the 80%, 10%, and 10% train, development, and test splits for the [SFU dataset](./data/auxiliary_tasks/en/SFU.conll) run the following bash script which will create `SFU_train.conll`, `SFU_dev.conll`, and `SFU_test.conll` files within [`./data/auxiliary_tasks/en/`](./data/auxiliary_tasks/en/) directory:
``` bash
./scripts/sfu_data_splits.sh
```

## Negation and Speculation Dataset Statistics

The table below states the complete dataset statistics, if the dataset like Conan Doyle has specified train, development, and test splits these are all combined and the combined statistics are stated below.

| Dataset     | Label       | No. Cues (tokens) | No. Scopes (tokens) | No. Sentences | No. Label Sentences |
|-------------|-------------|-------------------|---------------------|---------------|---------------------|
| SFU         | Negation    | 1,263 (2,156)     | 1,446 (8,215)       | 17,128        | 1,165               |
| SFU         | Speculation | 513 (562)         | 586 (3,483)         | 17,128        | 405                 |
| Conan Doyle | Negation    | 1,197 (1,222)     | 2,220 (9,761)       | 1,221         | 1,221               |

No. = Number

No. Label Sentences = Number of sentences from all of the sentences that contain at least one negation/speculation cue or scope token.

The number of scopes and cues states the number of complete BIO label spans where as the tokens defines the number of individual labels for instance [example 1](#example-1) from above contains 2 cues, 2 cue tokens, 2 scopes, and 3 scope tokens.

To generate the data statistics in the table above run the following bash script:
``` bash
./scripts/negation_statistics.sh
```

### Dataset split statistics
The tables below states the dataset **split** statistics:
#### SFU Negation

| Split    | No. Cues (tokens) | No. Scopes (tokens) | No. Sentences | No. Label Sentences |
|----------|-------------------|---------------------|---------------|---------------------|
| Train    | 1,018 (1,749)     | 1,155 (6,562)       | 13,712        | 934                 |
| Dev      | 121 (198)         | 154 (861)           | 1,713         | 114                 |
| Test     | 124 (209)         | 137 (792)           | 1,703         | 117                 |
| Combined | 1,263 (2,156)     | 1,446 (8,215)       | 17,128        | 1,165               |

To generate the data for this table above run `./scripts/negation_split_statistics.sh SFU negation`

#### SFU Speculation

| Split    | No. Cues (tokens) | No. Scopes (tokens) | No. Sentences | No. Label Sentences |
|----------|-------------------|---------------------|---------------|---------------------|
| Train    | 390 (425)         | 446 (2,623)         | 13,712        | 309                 |
| Dev      | 58 (63)           | 66 (402)            | 1,713         | 45                  |
| Test     | 65 (74)           | 74 (458)            | 1,703         | 51                  |
| Combined | 513 (562)         | 586 (3,483)         | 17,128        | 405                 |

To generate the data for this table above run `./scripts/negation_split_statistics.sh SFU speculation`

#### Conan Doyle Negation

| Split    | No. Cues (tokens) | No. Scopes (tokens) | No. Sentences | No. Label Sentences |
|----------|-------------------|---------------------|---------------|---------------------|
| Train    | 821 (838)         | 1,507 (6,756)       | 842           | 842                 |
| Dev      | 143 (146)         | 284 (1,283)         | 144           | 144                 |
| Test     | 233 (238)         | 429 (1,722)         | 235           | 235                 |
| Combined | 1,197 (1,222)     | 2,220 (9,761)       | 1,221         | 1,221               |

To generate the data for this table above run `./scripts/negation_split_statistics.sh conandoyle negation`

## Baselines
### Negation 
#### Conan Doyle
Run: `python scripts/negation_baseline.py ./resources/model_configs/negation_conan_doyle_baseline.jsonnet`

Generates:
``` python
Best epoch 24
Best validation span f1 0.8431137724550396
Test span f1 result 0.847611827141724
```

With a Bi-LSTM with 2 layers:
``` python
Best epoch 14
Best validation span f1 0.8382526564344245
Test span f1 result 0.8369646882043076
```

Bi-LSTM with 2 layers not training the embedding
``` python
Best epoch 50
Best validation span f1 0.847980997624653
Test span f1 result 0.8593155893535621
```

#### SFU
Run: `python scripts/negation_baseline.py ./resources/model_configs/negation_sfu_baseline.jsonnet`

Bi-LSTM with 2 layers not training the embedding
``` python
Best epoch 17
Best validation span f1 0.6978417266186547
Test span f1 result 0.6840277777777279
```

### Targeted Sentiment
If we base the model roughly on the work of [Li et al. 2019](https://www.aaai.org/ojs/index.php/AAAI/article/view/4643) which is a Bi-LSTM with CRF tagger but the Bi-LSTM contains two layers.
On the laptop
``` python
Best epoch 20
Best validation span f1 0.5855513307984289
Test span f1 result 0.5526315789473184
```

When the Bi-LSTM only contains one layer:
``` python
Best epoch 13
Best validation span f1 0.5317460317459816
Test span f1 result 0.4922826969942635
```

Bi-LSTM with 2 layers but not training the embeddings
``` python
Best epoch 45
Best validation span f1 0.581967213114704
Test span f1 result 0.561551433389495
```
Second run of the above:
```
Best epoch 21
Best validation span f1 0.5594989561586139
Test span f1 result 0.5358361774743531
```
Bi LSTM with 2 layers where the second layer has a skip connection from the word embedding
```
Best epoch 53
Best validation span f1 0.5968379446639813
Test span f1 result 0.5704918032786386
```
Second run of the above
```
Best epoch 47
Best validation span f1 0.593625498007918
Test span f1 result 0.5468227424748665
```

To generate the above results run: `python scripts/targeted_sentiment_baseline.py ./resources/model_configs/targeted_sentiment_laptop_baseline.jsonnet`

On Restaurant

Bi-LSTM with 2 layers but not training the embeddings
``` python
Best epoch 30
Best validation span f1 0.6232558139534383
Test span f1 result 0.6484342379957747
```
Bi-LSTM with 2 layers but not training the embeddings, skip connections between layer 1 and 2
```
Best epoch 39
Best validation span f1 0.6419161676646206
Test span f1 result 0.663265306122399
```
Run 2
```
Best epoch 22
Best validation span f1 0.6265356265355764
Test span f1 result 0.6460081773186503
```

To generate the above results run: `python scripts/targeted_sentiment_baseline.py ./resources/model_configs/targeted_sentiment_restaurant_baseline.jsonnet`

### Multi Task Learning
In the Multi task learning setup each epoch involves first training the negation model for one epoch and then training the sentiment model for an epoch. This is repeated until early stopping is applied based on the sentiment model score. NOTE that the multi task framework can support any number of additional task.

#### Hyperparameter tuning
Before running any of the experiments we first perform some hyperparameter tuning. The tuning is performed on the smallest datasets of both tasks which are the Laptop dataset for the Sentiment/Main task and the Conan Doyle for the Negation/Auxiliary task. The parameters we tune for are the following:
1. main/aux dropout rate
2. hidden size for shared encoder
3. batch_size 
4. starting learning rate for adam

Run the following:
``` bash
allentune search \
    --experiment-name multi_task_laptop_conan_search \
    --num-cpus 5 \
    --num-gpus 1 \
    --cpus-per-trial 5 \
    --gpus-per-trial 1 \
    --search-space resources/tuning/tuning_configs/multi_task_search_space.json \
    --num-samples 30 \
    --base-config resources/tuning/tuning_configs/multi_task_laptop_conan.jsonnet \
    --include-package multitask_negation_target
allentune report \
    --log-dir logs/multi_task_laptop_conan_search/ \
    --performance-metric best_validation_f1-measure-overall \
    --model multi-task
allentune plot \
    --data-name Laptop \
    --subplot 1 1 \
    --figsize 10 10 \
    --result-file logs/multi_task_laptop_conan_search/results.jsonl \
    --output-file tuning/multi_task_tuning_laptop_performance.pdf \
    --performance-metric-field best_validation_f1-measure-overall \
    --performance-metric F1-Span
```

#### Example of how to run the Multi-Task System
You can use the allennlp train command here:
```
allennlp train resources/model_configs/multi_task_trainer.jsonnet -s /tmp/any --include-package multitask_negation_target
```

## Cross domain (Ignore this)
Idea on the cross domain would be to train the negation model and then apply it to some texts within the target domain and then train on those texts, similar type of idea to Semi-Supervised Learning

Train on laptop with negation as auxilary and then test on Restaurant
`python scripts/multi_task_baseline.py ./resources/model_configs/transfer_conan_laptop_restaurant_shared_baseline.jsonnet`
```
Best epoch 51
Negation Results
Validation F1 measure: 0.8419782870928328
Test F1 measure: 0.849230769230719
Sentiment Results
Validation F1 measure: 0.6309278350514963
Test F1 measure: 0.1962209302325301
```


Train on laptop and test on restaurant
`python scripts/targeted_sentiment_baseline.py ./resources/model_configs/targeted_sentiment_laptop_restaurant_baseline.jsonnet`
```
Best epoch 34
Best validation span f1 0.581443298969022
Test span f1 result 0.19215545160126632
```

## Requirements

1. Python >= 3.6
2. `pip install .` 
3. If you are developing or want to run the tests `pip install -r requirements.txt`
3. sklearn  ```pip install -U scikit-learn```
4. Pytorch ```pip install torch torchvision```

## Run the tests:
`python -m pytest`

## Other ideas
1. Having a shared and task specific Bi-LSTM layers at the moment all Bi-LSTM layers are shared
2. The affect of having very little sentiment data and all of the negated data
3. The affect of pre-trained word embeddings, if random does multi-task/transfer help a lot more.