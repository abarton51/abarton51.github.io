# RAG Paper Notes

Paper Title: **Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks**

### Abstract
- Large pre-trained language models have been shown to store factual knowledge in their parameters.
- Downfall is that their ability to access and manipulate knowledge is limited.
- Struggle or fail to provide provenance for their decisions and updating their world knowledge.
- Present a general-purpose fine-tuning recipe for retrieval-augmented generation (RAG).
	- Models which combine pre-trained parametric and non-parametric memory for language generation.
- Parametric memory is a pre-trained seq2seq model.
- Non-parametric memory is a dense vector index of Wikipedia, accessed with a pre-trained neural retriever.

### Introduction
- Pre-trained neural language models learn substantial amount of in-depth knowledge from data.
- Cannot easily expand or revise their memory.
- Can't straightforwardly provide insight into their predictions, and may produce hallucinations.
- Hybrid models that combine parametric memory with non-parametric memories can address some of these issues because knowledge can be directly revised and expanded, and accessed knowledge can be inspected and interpreted.
	- REALM and ORQA combine masked language models with a differentiable retriever.
	- Have only explore open-domain extractive question answering.
	- This paper brings hybrid memory to seq2seq models.
- Endow pre-trained, parametric-memory generation models with a non-parametric memory through a general-purpose fine-tuning approach which they refer to as retrieval-augmented generation (RAG).
- Parametric memory is a pre-trained seq2seq transformer.
- Non-parametric memory is a dense vector index of Wikipedia, accessed with a pre-trained neural retriever.
- Combine parametric and non-parametric memory in a probabilistic model trained end-to-end.
- The retriever provides latent documents conditioned on the input
	- Dense Passage Retriever (DPR)
- The seq2seq model then conditions on these latent documents together with the input to generate the output.
	- BART
- Marginalize the latent docs with a top-K approximation.
	- Done on either a per-output basis or a per-token basis.
- RAG can be fine-tuned on any seq2seq task, whereby both the generator and the retriever are jointly learned.

### Methods
- Use the input sequence $x$ to retrieve text documents $z$ and use them as additional context when generating the target sequence $y$.
- Models leverage two components
1. A retriever $p_{\eta}(z|x)$ with parameters $\eta$ that returns (top-K truncated) distributions over text passages given a query $x$
2. A generator $p_{\theta}(y_i|x, z, y_{1:i-1})$ parametrized by $\theta$ that generates a current token based on a context of the previous $i-1$ tokens $y_{1:i-1}$, the original input $x$ and a retrieved passage $z$.

- To train the retriever and generator end-to-end, we treat the retrieved document as a latent variable. They propose two models that marginalize over the latent documents in different ways to produce a distribution over generated text.
	- In one approach, RAG-Sequence, the model uses the same document to predict each target token.
	- In the second approach, RAG-Token, can predict each target token based on a different document.

#### 2.1 Models
- **RAG-Sequence**: Uses the same retrieved document to generate the complete *sequence*. Treats the retrieved document as a single latent variable that is marginalized to get the seq2seq probability $p(y|x)$ via a top-K approximation. 
	- The top $K$ documents are retrieved using the retriever, and the generator produces the output sequence probability for each document, which are then marginalized,
$$p_{\text{RAG-Sequence}}(y|x) \approx \sum_{z\in \text{top-k}(p(\cdot | x))} p_{\eta}(z|x)p_{\theta}(y|x, z) = \sum_{z\in \text{top-k}(p(\cdot | x))} p_{\eta}(z|x)\prod_i^N p_\theta (y_i|x, z, y_{1:i-1})$$
- **RAG-Token**: Can draw a different latent document for each target *token* and marginalize accordingly. 
	- This allows the generator to choose content from several documents when producing an answer.
	- The top $K$ documents are retrieved using the retriever, and then the generator produces a distribution for the next output token for each document, before marginalizing, and repeating the process with the following output token.
$$p_{\text{RAG-Token}}(y|x) \approx \prod_{i}^N \sum_{z\in \text{top-k}p(\cdot|x)} p_\eta(z|x)p_\theta(y_i|x, z, y_{1:i-1})$$
- Note that RAG can be used for sequence classification tasks by considering the target class as a target sequence of length one, in which case RAG-Sequence and RAG-Token are equivalent.

#### 2.2 Retriever: DPR
The retrieval component $p_{\eta}(z|x)$ is based on DPR. DPR follows a bi-encoder architecture:
$$ p_\eta(z|x)\propto \exp(d(z)^T q(x)) $$
where $d(z) = \text{BERT}_d(z), q(x) = \text{BERT}_q(x)$. 
$d(z)$ is a dense representation of a document produced by a $\text{BERT}_\text{BASE}$ document encoder and $q(x)$ is a query representation produced by a query encoder, also based on $\text{BERT}_\text{BASE}$.

Calculating the $\text{top-k}(p_\eta(z|x))$, the list of $k$ documents $z$ with highest prior probability $p_\eta(z|x)$, is a Maximum Inner Product Search (MIPS) problem, which can be approximately solved in sub-linear time. They used a pre-trained bi-encoder from DPR to initialize the retriever and to build the document index.

- Non-parametric memory refers to the document index.

#### 2.3 Generator: BART
The generator component $p_\theta(y_i|x, z, y_{1:i-1})$ could be modeled using any encoder-decoder but they use BART-large, a pre-trained seq2seq transformer with 400M parameters.

- To combine the input $x$ with the retrieved content $z$ when generating BART, they simply concatenate them.
- Parametric memory refers to the BART generator.

#### 2.4 Training
Jointly train the retriever and generator components without any direct supervision on what document should be retrieved.

Given a fine-tuning training corpus of input/output pairs $(x_j, y_j)$, they minimize the negative marginal log-likelihood of each target, $\sum_j - \log p(y_j|x_j)$ using SGD with Adam optimizer.

- Updating the document encoder $\text{BERT}_d$ during training is costly as it requires the document index to be periodically updated as REALM does during pre-training.
- They don't find this to be necessary and keep the document encoder and index fixed, only fine-tuning the query encoder $\text{BERT}_q$ and the BART generator.

#### 2.5 Decoding
At test time, RAG-Sequence and RAG-Token require different ways to approximate $\arg \max_y p(y|x)$.

- **RAG-Token**: The RAG-Token model can be seen as a standard, autoregressive seq2seq generator with transition probability: $p_\theta'(y_i|x, y_{1:i-1})$. To decode, we can plug this probability into a standard beam search decoder.
- **RAG-Sequence**: For RAG-Sequence, the likelihood $p(y|x)$ does not break into a conventional per-token likelihood, hence we can't solve it with a single beam search.
	- Instead we run beam search for each document $z$, scoring each hypothesis using $p_\theta(y_i|x, y_{1:i-1})$. This yields a set of hypotheses $Y$, some of which may not have appeared in the beams of all documents.
	- To estimate the probability of a hypothesis $y$, they run an additional forward pass for each document $z$ for which $y$ does not appear in the beam, multiply the generator probability with $p_\eta(z|x)$ and then sum the probabilities across beams for the marginals.
	- This is called "Thorough Decoding". 



