---
title: "JMM '24 - Things I Learned"
excerpt: "Notes on Interesting Talks at JMM '24."
collection: portfolio
---

I attended the 2024 Joint Mathematics Meeting to present a poster on research I did as part of an REU at NC State University. You can check out the research in the navigation bar.

The time I spent there with my friends and peers from the previous summer was amazing. Mathematicians from all over the world gather there to present the most cutting edge research in mathematics to their peers and bright-eyed students like myself. While I was there, I took some notes on the things I learned.

# Things I Learned About
## VC Dimension
The **Vapnik-Chervonenkis dimension**, more commonly known as the VC dimension, is a model capacity measurement used in statistics and machine learning. It is termed informally as a measure of a model’s capacity. It is used frequently to guide the model selection process while developing machine learning applications. To understand the VC dimension, we must first understand shattering. [1](https://www.educative.io/answers/what-is-the-vc-dimension) 
### Shattering
**Shattering** is the ability of a model to classify a set of points perfectly. More generally, the model can create a function that can divide the points into two distinct classes without overlapping. It is different from simple classification because it considers all possible combinations of labels upon those points. In the context of shattering, we simply define the VC dimension of a model as the size of the largest set of points that that model can shatter.

Consider the binary classification setup in which we’re given some data from the domain $\mathcal{X}$, and we’d like to predict $0$ or $1$ for each data point. Specifically, consider a sample $\mathcal{C}\subseteq \mathcal{X}$ containing $n$ data points. Because $n$ is finite, there are only a finite number of ways to possibly label the data; specifically, there are $2^n$ labelings.

We say that a hypothesis class $\mathcal{H}$ _shatters_ $\mathcal{C}$ if it’s able to represent all $2^n$ of these functions. Said another way, if we’re given any possible labeling of $\mathcal{C}$, by using hypotheses within $\mathcal{H}$ we will always be able to find a hypothesis that achieves zero error.

- The VC dimension of a hypothesis class $\mathcal{H}$ is the size of the largest (sample) set $\mathcal{C}$ that $\mathcal{H}$ is able to shatter. [2](https://andrewcharlesjones.github.io/journal/vc-dimension.html#:~:text=VC%20dimension%20is%20a%20measure,through%20a%20couple%20simple%20examples.) 

In practice computing the VC dimension will be more complex (think about the expressiveness of neural networks, for example).

It’s also possible for the VC dimension to be infinite. In fact, if this is the case, then it turns out that $\mathcal{H}$ is not PAC learnable. This should intuitively make sense, because an infinite VC dimension implies that it will be very difficult to search over the set of hypotheses, and the class is _so_ expressive that it’s basically meaningless.

## PAC Learning
**Probably approximately correct learning** is a framework for mathematical analysis of machine learning.

In this framework, the learner receives samples and must select a generalization function (called the *hypothesis*) from a certain class of functions. The goal is that, with high probability, the selected function will have low generalization error. The learner must be able to learn the concept given any arbitrary approximation ratio, probability of success, or distribution of samples.

In order to give the definition for something that is PAC-learnable, we first have to introduce some terminology. [3](https://en.wikipedia.org/wiki/Probably_approximately_correct_learning#cite_note-2)

For the following definitions, two examples will be used. The first is the problem of [character recognition](https://en.wikipedia.org/wiki/Character_recognition "Character recognition") given an array of $n$ bits encoding a binary-valued image. The other example is the problem of finding an interval that will correctly classify points within the interval as positive and the points outside of the range as negative.

Let $\mathcal{X}$ be a set called the _instance space_ or the encoding of all the samples. In the character recognition problem, the instance space is $\mathcal{X} = \{0, 1\}^n$. In the interval problem, the instance space, $\mathcal{X}$, is the set of all bounded intervals in $\mathbb{R}$.

A _concept_ is a subset $c\subseteq \mathcal{X}$. One concept is the set of all patterns of bits in $\mathcal{X} = \{0, 1\}^n$ that encode a picture of the letter "P". An example concept from the second example is the set of open intervals,$\{ (a, b)| 0 \leq a \leq \pi/2, \pi \leq b \leq \sqrt{13}  \}$, each of which contains only the positive points. A _[concept class](https://en.wikipedia.org/wiki/Concept_class "Concept class")_ $\mathcal{C}$ is a collection of concepts over $\mathcal{X}$. This could be the set of all subsets of the array of bits that are [skeletonized](https://en.wikipedia.org/wiki/Morphological_skeleton "Morphological skeleton") [4-connected](https://en.wikipedia.org/wiki/Pixel_connectivity#4-connected "Pixel connectivity") (width of the font is 1).

Let $EX(c, D)$ be a procedure that draws an example, $x$, using a probability distribution $D$ and gives the correct label $c(x)$, that is 1 if $x\in c$ and $0$ otherwise.

Now, given $0 < \epsilon$, $\delta < 1$, assume there is an algorithm $A$ and a polynomial $p$ in $1/\epsilon$, $1/\delta$ such that, given a sample size of $p$ drawn according to $EX(c, D)$, then, with probability at least $1-\delta$, $A$ outputs a hypothesis $h\in C$ that has an average error less than or equal to $\epsilon$ on $\mathcal{X}$ with the same distribution $D$. Further, if the above statement for algorithm $A$ is true for every concept $c\in C$ and for every distribution over $\mathcal{X}$, and for all $0< \epsilon$, $\delta < 1$, then $C$ is (efficiently) **PAC learnable**. We can also say that $A$ is a **PAC learning algorithm** for $C$.
