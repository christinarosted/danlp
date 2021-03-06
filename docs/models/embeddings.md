Pretrained Danish embeddings
============================
This repository keeps a list of pretrained word embeddings publicly available in Danish. The `download_embeddings.py`
and `load_embeddings.py` provides functions for downloading the embeddings as well as prepare them for use in 
popular NLP frameworks.

| Name | Model | Data | Vocab | Unit | Task  | Pretrainer |
|------|-------|------|:-----:|------|-------|---------------|
| [connl.da.wv](http://vectors.nlpl.eu/repository/#) | word2vec | [Danish CoNLL17](http://universaldependencies.org/conll17/) | 1.655.886 | Word | Skipgram | [University of Oslo](https://www.mn.uio.no/ifi/english/) |
| [news.da.wv](https://loar.kb.dk/handle/1902/329) | word2vec | OCR Newspapers 1880-2005 | 2.404.836 | Word | Skipgram | [Det Kgl. Bibliotek](http://www.kb.dk) |
| [cc.da.swv](https://fasttext.cc/docs/en/crawl-vectors.html) | fastText | CC + Wiki | 2.000.000 | Char N-gram | Skipgram |  [Facebook AI Research](https://research.fb.com/category/facebook-ai-research/) |
| [wiki.da.swv](https://fasttext.cc/docs/en/pretrained-vectors.html)| fastText | Wikipedia | 312.956 | Char N-gram | Skipgram | [Facebook AI Research](https://research.fb.com/category/facebook-ai-research/) |
| forward_embedding backward_embedding | Flair | Wikipedia + Europarl | | Char | LM | [Alexandra Institute](https://alexandra.dk/uk) |

Embeddings are a way of representing text as numeric vectors, and can be calculated both for chars, subword units [(Sennrich et al. 2016)](https://aclweb.org/anthology/P16-1162), 
words, sentences or documents.
The methods for training embeddings, can roughly be categorized into static embeddings and dynamic embeddings.

#### Static embeddings
Static word embeddings contains a large vocabulary of words and each word has a vector representation associated.
To get a representation of a word is simply a look-up in the vocabulary to get the associated vector. An example of this
type of embeddings is word2vec [(Mikolov et al. 2013)](https://papers.nips.cc/paper/5021-distributed-representations-of-words-and-phrases-and-their-compositionality.pdf).
Relying on a vocabulary of words can result in out-of-vocabulary words. To cope with this fastText [(Bojanowski et al. 2017)](https://aclweb.org/anthology/Q17-1010)
uses subword units that constructs a word embedding from the character n-gram embeddings occurring in the word.

#### Dynamic embeddings
Dynamic embeddings are contextual in the sense that the representations are dependent on the sentence they appear in.
This way homonyms get different vector representations. An example of dynamic embeddings is the Flair embeddings [(Akbik et al. 2018)](https://aclanthology.coli.uni-saarland.de/papers/C18-1139/c18-1139)
where the embeddings are trained with the task of language modelling ie. learning to predict 
the next character in a sentence.

## Using word embeddings for analysis

Word embeddings are essentially a representation of a word in a n-dimensional space.
Having a vector representation of a word enables us to find distances between words.
In `load_embeddings.py` we have provided functions to download pretrained word embeddings and load them with
the two popular NLP frameworks [spaCy](https://spacy.io/) and [Gensim](https://radimrehurek.com/gensim/).

This snippet shows how to automatically download and load pretrained word embeddings e.g. trained on the CoNLL17 dataset.
```python
from danlp.models.embeddings  import load_wv_with_gensim, load_wv_with_spacy

# Load with gensim
word_embeddings = load_wv_with_gensim('connl.da.wv')

word_embeddings.most_similar(positive=['københavn', 'england'], negative=['danmark'], topn=1)
# [('london', 0.7156291604042053)]

word_embeddings.doesnt_match("vand sodavand brød vin juice".split())
# 'brød'

word_embeddings.similarity('københavn', 'århus')
# 0.550142

word_embeddings.similarity('københavn', 'esbjerg')
# 0.48161203


# Load with spacy
word_embeddings = load_wv_with_spacy('connl.da.wv')

```

## Pretrained Flair embeddings
Thus repository provides pretrained Flair word embeddings trained on Danish data from Wikipedia and EuroParl
both forwards and backwards. To see the code for training the Flair embeddings have a look at  [Flairs GitHub](https://github.com/zalandoresearch/flair).

The hyperparameter are set as follows: `hidden_size=1032`, `nlayers=1`, `sequence_length=250`, `mini_batch_size=50`, 
`max_epochs=5`

In the snippet below you can see how to load the pretrained Danish embeddings and an example of simple use. 

```python
from danlp.models.embeddings import load_context_embeddings_with_flair
from flair.data import Sentence

# Use the wrapper from DaNLP to download and load embeddings with Flair
stacked_embeddings = load_context_embeddings_with_flair()

# Embed two different sentences
sentence1 = Sentence('Han fik bank')
sentence2 = Sentence('Han fik en ny bank')
stacked_embeddings.embed(sentence1)
stacked_embeddings.embed(sentence1)

# Show that it is contextual in the sense 'bank' has different embedding after context
print('{} sentences out of {} is equal'.format(int(sum(sentence2[4].embedding==sentence1[2].embedding)), len(sentence1[2].embedding)))
# 52 ud af 2364
```


The trained Flair word embeddings has been used in training the Danish Part of speech model with Flair, check it out [here](<https://github.com/alexandrainst/danlp/blob/master/docs/models/pos.md>). 



## 📈 Benchmarks

To evaluate word embeddings it is common to do intrinsic evaluations to directly test for syntactic or 
semantic relationships between words. The WordSimilarity-353 dataset [(Finkelstein et al. 2002)](http://www.cs.technion.ac.il/~gabr/papers/tois_context.pdf)
contains word pairs annotated with a similarity score (1-10) and calculating the correlation between 
the word embedding similarity and the similarity score gives an indication of how well the word embeddings
captures relationships between words. The dataset has been 
[translated to Danish](https://github.com/fnielsen/dasem/tree/master/dasem/data/wordsim353-da) by Finn Aarup Nielsen. 

| Model | Spearman's rho | OOV |
| ------:|:----------------:|:----------:|
| cc.da.wv | **0.5917** | 0% |
| wiki.da.wv | 0.5851 | 5.01% |
| connl.da.wv | 0.5243 | 5.01% |
| news.da.wv | 0.4961 | 5.6% |

## 🎓 References
- Thomas Mikolov, Ilya Sutskever, Kai Chen, Greg Corrado and Jeffrey Dean. 2013. [Distributed Representations of Words and Phrasesand their Compositionality](https://papers.nips.cc/paper/5021-distributed-representations-of-words-and-phrases-and-their-compositionality.pdf). In **NeurIPS**.
- Piotr Bojanowski, Edouard Grave, Armand Joulin and Tomas Mikolov. 2017. [Enriching Word Vectors with Subword Information](https://aclweb.org/anthology/Q17-1010). In **ACL**.
- Rico Sennrich, Barry Haddow and Alexandra Birch. 2016. [Neural Machine Translation of Rare Words with Subword Units](https://aclweb.org/anthology/P16-1162). In **ACL**.
- Lev Finkelstein, Evgeniy Gabrilovich, Yossi Matias, Ehud Rivlin, Zach Solan, Gadi Wolfman, and Eytan Ruppin. 2002. [Placing Search in Context: The Concept Revisited](http://www.cs.technion.ac.il/~gabr/papers/tois_context.pdf). In  **ACM TOIS**.
- Alan Akbik, Duncan Blythe and Roland Vollgraf. 2018. [Contextual String Embeddings for Sequence Labeling](https://aclanthology.coli.uni-saarland.de/papers/C18-1139/c18-1139). In **COLING**.