## Transfer-Based Machine Translation
- Idea: improve rule-based MT
	1. source analysis: analyze source language → build syntactic model for source text
	2. transfer: source language parse tree → target language parse tree
	3. generation: target language parse tree → output sentence
- transfer stage is still rule-based, but rules on syntactic structures are more generalizable
	- POS + a set of rules based on grammar of source and target language (SOV vs SVO)
	- long sentence reordering is easier
	- SYSTRAN (Peter Toma, 1968) systems are based on this approach
		![[Pasted image 20231115143532.png]]
## Statistical Machine Translation
- *Parallel texts (corpora)*
	- sets of documents where each document in 1 language is accompanied by its translation in another language
	- examples:
		- Canadian Hansards, French-English documents
			- 1.7 million sentences of 30 words or less in length
		- European Union
		- translations of books
	- Idea: use [[parallel corpora]] as training set of translation examples
		- train translation model, relaxing the need for rules
- the primary machine translation model is the **Noisy Channel** model
	- train model that receives a sentence in source language → returns sentence in target language
	- famous example: IBM Models in the 90s
### Noisy Channel Model for MT
- it's called the *noisy channel* because the model imagines that the source sentence is the *original message* and was sent through a noisy channel and received as the target sentence
	- Goal: find the most probably source sentence given the target sentence
- for this class assume the primary goal is to translate a text from French to English
- Idea: generate a model $p(e|f)$ which estimates the conditional probability of any English sentence $e$ given the French sentence $f$
	- use training corpus to set model parameters
- Definition:
	$\quad p(e|f) = \frac{p(e,f)}{p(f)} = \frac{p(e)p(f|e)}{\sum_{e}p(e)p(f|e)}$ where
	$\qquad p(e):$ the *language* model
	$\qquad p(f|e):$ the *translation* model
- then the decoding problem is defined by
	$\quad\text{argmax}_{e}p(e|f) = \text{argmax}_{e}p(e)p(f|e)$
### Language Model: [[Trigram]]
- Example: Spanish to English
	![[Pasted image 20231115143611.png]]
### Translation Model: IBM Model
- Idea: in the parallel corpus, consider that, for a pair, the English sentence has $l$ words and the French sentence has $m$ words.
	- an *alignment map* determines which English word that each French word originated from
		- an alignment $a$ is $\{ a_{1},\dots a_{m} \}$, where $a_j\in\{ 0\dots l\}$
		- hence, there are $(l+1)^m$ possible alignments
		![[Pasted image 20231115143639.png]]
- the total probability over all possible alignments is defined by
	$\quad p(f|e,m) = \sum_{a\in A} p(a|e,m) p(f|a,e,m)$
	- we then model the conditional probabilities $p(a|e,m)$ and $p(f|a,e,m)$
	- we need to compute these 2 probabilities using the parallel corpus
#### IBM Model 1
- a possible model is assigning equally likely alignment probability
	$\quad p(a|e,m) = \frac{1}{(l+1)^m}$
- and the conditional translation model is
	$\quad p(f|a,e,m) = \prod_{j=1}^m t(f_{j|e_{a_{j}}})$
- Example calculation
	$\quad l=6, m=7$
	$\quad a = \{ 2,3,4,5,6,6,6 \}$
	$\quad e =$ *And the program has been implemented*
	$\quad f=$ *Le programme a ete mis en application*
	$\qquad p(f|a,e) = t(Le|the) \times t(programme|program) \times t(a|has)$
	$\qquad \qquad  \times\, t(ete|been) \times t(mis|implemented)$
	$\qquad \qquad \times\, t(en|implemented) \times t(application|implemented)$
- <span style="color:#f55656">Example model paremeters</span>

	| English  | French    | Probability |
	| -------- | --------- | ----------- |
	| position | position  | 0.7567      |
	| position | situation | 0.0547      |
	| position | mesure    | 0.0281      |
	| position | vue       | 0.0169      |
	| position | point     | 0.0124      |
	| position | attitude  | 0.0108      |

- Generative process:
	1. pick an alignment randomly
		$\quad \frac{1}{(l+1)^m}$
	2. pick the corresponding French words
		$\quad p(f|a,e,m) = \prod_{j=1}^mt(f_{i}|e_{a_{j}})$
	3. compute conditional translation probability
		$\quad p(f,a|e,m) = p(a|e,m)\times p(f|a,e,m) = \frac{1}{(l+1)^m}\prod_{j=1}^mt(f_{j}|e_{a_{j}})$
#### IBM Model 2
- non-uniform alignments: distortion parameters
	- Model 1
		$\quad p(a|e,m) = \frac{1}{(l+1)^m}$
	- Model 2
		$\quad p(a|e,m) = \prod_{j=1}^m q(a_{j=i}|j,l,m)$ where
		$\qquad j$'s French word is generated from $i$'s English word given the lengths
- Conditional Translation model
	- Model 1
		$\quad p(f|a,e,m) = \prod_{j=1}^m t(f_{i|e_{a_{j}}})$
	- Model 2
		$\quad p(f|a,e,m) = \prod_{j=1}^m q(a_{j}|j,l,m)t(f_{j}|e_{a_{j}})$
	- Example
		$\quad l=6$
		$\quad m=7$
		$\quad e=$ *And the program has been implemented*
		$\quad l=$ *Le programme a ete mis en application*
		$\quad l= \{ 2,3,4,5,6,6,6 \}$
		$\qquad p(a|e,7) = q(2|1,6,7) \times q(3|2,6,7) \times q(4|3,6,7) \times$
		$\qquad \quad q(5|4,6,7) \times q(6|5,6,7) \times q(6|6,6,7) \times q(6|7,6,7)$
		$\qquad p(f|a,e,7) = t(Le|the) \times t(programme|program) \times t(a|has) \times t(ete|been) \times$
		$\qquad \quad t(mis|implemented) \times t(en|implemente) \times t(application|implemented)$
	- Generative process:
		1. pick alignment randomly
			$\quad \prod_{j=1}^mq(a_{j}|j,l,m)$
		2. pick corresponding French words
			$\quad p(f|a,e,m) = \prod_{j=1}^mt(f_{j}|e_{a_{j}})$
		3. compute conditional translation probability
			$\quad p(f,a|e,m) = p(a|e,m)p(f|a,e,m) = \prod_{j=1}^mq(a_{j}|j,l,m)t(f_{i},e_{a_{j}})$
#### IBM Model Parameter Estimation
- input: sentence pairs $(e^{(k)}, f^{(k)})$
- output: parameters $t(f|e)$ and $q(i|j,l,m)$
- primary challenge: alignments are not known
	- data annotation is expensive
		$\quad e^{(100)} =$ And the program has been implemented
		$\quad f^{(100)} =$ Le programme a ete mis en application
	- chicken and egg problem
- Expectation Maximization (EM) algorithm
##### Idea
1. assume alignments are accessible
	$\quad e^{(100)} =$ *And the program has been implemented*
	$\quad f^{(100)} =$ *Le programme a ete mis en application*
	$\quad a^{(100)}= \{ 2,3,4,5,6,6,6 \}$
2. we will then have triplets $(e^{(k)}, f^{(k)}, a^{(k)})$
3. now, model estimates for parameters boils down to counting, ex, $t(position|position)$
	$\quad t_{ML}(f|e) = \frac{Count(e,f)}{Count(e)}$
	$\quad q_{ML}(j|i,l,m) = \frac{Count(j,i,l,m)}{Count(i,l,m)}$
##### Execution
**Input**
Given a training corpus $(f^{(k)}, e^{(k)}, a^{(k)})$ for $k=1\dots n$ where
$\quad f^{(k)}=f_{1}^{(k)}\dots f_{mk}^{(k)}$
$\quad e^{(k)} = e_{1}^{(k)}\dots e_{e_{lk}}^{(k)}$
$\quad a^{(k)} = a_{1}^{(k)}\dots a_{a_{mk}}^{(k)}$
**Algorithm**
set all counts $c(\dots) = 0$
for $k=1\dots n:$
$\quad$ for $i=1\dots m_{k}:$
$\qquad$ for $j=0…l_{k}:$
	$\qquad \quad c(e_{j}^{(k)}), f_{i}^{(k)} \leftarrow c(e_{j}^{(k)}, f_{i}^{(k)}) + \delta(k,i,j)$
	$\qquad \quad c(e_{j}^{(k)}) \leftarrow c(e_{j}^{(k)} + \delta(k,i,j))$
	$\qquad \quad c(j|i,l,m) \leftarrow c(j|i,l,m) + \delta(k,i,j)$
	$\qquad \quad c(i,l,m) \leftarrow  c(i,l,m) + \delta(k,i,j)$
$\qquad$where
	$\qquad \quad \delta(k,i,j) = 1$ if $a_{i}^{(k)} = j$, 0 otherwise
**Output**
We obtain the MLE of translation probability $t_{ML}$ and the MLE of alignment probability $q_{ML}$, where
$\quad t_{ML}(f|e) = \frac{c(e,f)}{c(e)}$
$\quad q_{ML}(j|i,l,m) = \frac{c(j|i,l,m)}{c(i,l,m)}$
## [[Expectation Maximization]]
### Introduction
- algorithm invented by Dempster et al. 1977 for computing MLE from incomplete data
	- *incomplete data:* data annotation absent for estimating model parameters
	- if we did have complete data, we could estimate the model easily
	- if we had the model, we could fill in gaps in the data
- in a nutshell:
	1. random initialization of model parameters
	2. assign probabilities to missing data (E-step)
	3. estimate model parameters from completed data (M-step)
	4. iterate steps 2-3 until convergence
### [[Algorithm]]
1. expectation-step: apply model to data and compute probability of alignments
	- parts of the model are hidden (alignments)
	- using the model, assign probabilities to possible values
2. maximization-step: estimate model from data through count collection
	- take assigned values as fact
	- collect counts (weighted by probabilities)
	- estimate model from counts
3. iterate on steps 1 and 2 until convergence
#### A few notes
- the algorithm is iterative; we start with some arbitrary choice for $q$ and $t$ parameters
- at each iteration we compute *counts* based on the data together with our current parameter estimates
- we then re-estimate the parameters with these counts and iterate
- the delta function $\delta(k,i,j)$ is defined by
	$\quad \delta(k,i,j) = \frac{q(j|i,l_{k},m_{k})t(f_{i}|e_{j}^{(k)})}{\sum_{j=0}^{l_{k}}q(j|i,l_{k},m_{k})t(f_{i}^{(k)}|e_{j}^{(k)})}$
- returning to the above formula
	set all counts $c(\dots) = 0$
	for $k=1\dots n:$
	$\quad$ for $i=1\dots m_{k}:$
	$\qquad$ for $j=0…l_{k}:$
	$\qquad$ **M-step**
	$\qquad \quad c(e_{j}^{(k)}), f_{i}^{(k)} \leftarrow c(e_{j}^{(k)}, f_{i}^{(k)}) + \delta(k,i,j)$
	$\qquad \quad c(e_{j}^{(k)}) \leftarrow c(e_{j}^{(k)} + \delta(k,i,j))$
	$\qquad \quad c(j|i,l,m) \leftarrow c(j|i,l,m) + \delta(k,i,j)$
	$\qquad \quad c(i,l,m) \leftarrow c(i,l,m) + \delta(k,i,j)$
	$\qquad$ **E-step**
	$\qquad \quad \delta(k,i,j) = \frac{q(j|i,l_{k},m_{k})t(f_{i}|e_{j}^{(k)})}{\sum_{j=0}^{l_{k}}q(j|i,l_{k},m_{k})t(f_{i}^{(k)}|e_{j}^{(k)})}$
$\quad$ and we recalculate the parameters
$\qquad t_{ML}(f|e) = \frac{c(e,f)}{c(e)}$
$\qquad q_{ML}(j|i,l,m) = \frac{c(j|i,l,m)}{c(i,l,m)}$
#### Visualization
![[Pasted image 20231115143703.png]]
![[Pasted image 20231115143718.png]]
### Model Evaluation
- we use *[[perplexity]]*: deriving the probability of the training data according to the model
	$\quad \log_{2}PP = -\sum_{s}\log_{2}p(e_{s}|f_{s})$
- Example

	|                        | initial | iter 1 | iter 2 | iter 3 | …   | final  |
	| ---------------------- | ------- | ------ | ------ | ------ | --- | ------ |
	| p(the haus \ das hous) | 0.0625  | 0.1875 | 0.1905 | 0.1913 | …   | 0.1875 |
	| p(the book \ das huch) | 0.0625  | 0.1406 | 0.1790 | 0.2075 | …   | 0.25   |
	| p(a book \ ein buch)   | 0.0625  | 0.1875 | 0.1907 | 0.1913 | …   | 0.1875 |
	| perplexity             | 4095    | 202.3  | 153.6  | 131.6  | …   | 113.8  |
	
## Summary
- key ideas in IBM translation models
	1. alignment variables
	2. translation parameters e.g. $t(chien|dog)$
	3. distortion parameters e.g. $q(2|1,6,7)$
- EM algorithm: iterative algorithm for training the $q$ and $t$ parameters
	- once parameters are trained, we can recover the most likely alignments on our training samples
- Weakness of IBM model alignments:
	- noisy → not accurate
	- many-to-one: many words in source language can be mapped to a single word
		- but for 1 source word we find 1 target word
	- non-compositional phrases are not encoded
	- context not considered in translation
	- propositions may not be translated properly
### Phrase Based Translation Models
- next up is *phrase based translation models*
- Idea:
	- word-based models translate **words** as atomic atomic units
	- phrase-based models translate **phrases** as atomic units
- Advantages:
	- many-to-many translations can handle non-compositional phrases e.g. red herring, hot dog
	- use of local context in translation
	- the more the data, the longer the learned phrases
- state of the art, used by Google Translate until 2017
