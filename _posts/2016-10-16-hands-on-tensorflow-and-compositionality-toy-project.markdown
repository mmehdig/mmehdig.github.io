---
layout: post
title:  Hands-on TensorFlow and compositionality toy project
date:   2016-10-16 23:59:59 +0200
categories: blog
comments: true
---

Last week, I promised to have a blog post about my experiments on TensorFlow. I don't want to repeat whatever is out there about it like another tutorial but I don't want to make it unreadable for general audience who just want to know basics. Here it is what I will do: First a very brief review of TensorFlow (what and why, how), then I will give you a simplified version of my experiments as an example.


### Very brief review

**What is TensorFlow?** TensorFlow is a numerical computation tool with interfaces in python and C++.

**Why is it useful?** Its architecture has flexibility to perform computation on several CPUs or GPUs of any desktop or server.

**How does it work?** It has a concept of graph which has operations as its nodes and the inputs and outputs of these operations are directed edges of graph. Each line of code for a TensorFlow operation (e.g. multiplication of matrixes) adds a node in this graph. Then you can run a session and with in this session you can push your information and ask for outputs at any nodes of this graph. Let's see an example.

### A simple example (matrix multiplication)

Let's try [this matrix multiplication](https://www.youtube.com/watch?v=kT4Mp9EdVqs) in TensorFlow (in python):

{% highlight python %}
import tensorflow as tf
{% endhighlight %}

Here it is our graph:

{% highlight python %}
A = tf.placeholder(tf.float32, shape=(2, 2))
B = tf.placeholder(tf.float32, shape=(2, 2))
AB = tf.matmul(A, B)
{% endhighlight %}

The `tf.placeholder` basically define inputs with their size (`shape` 2 by 2) and types of float32. The last line, `tf.matmul` does the multiplication and assign it to `AB`. Now, we can fed it with matrixes and ask for results.

{% highlight python %}
sess = tf.Session()
result = sess.run(AB, {A: [[2, -2], [5, 3]], B: [[-1, 4], [7, -6]]})
sess.close()
{% endhighlight %}

Yes, as simple as this. Maybe this part is a bit strange for python programmers, `A` and `B` in session part are pointing at `A` and `B` in graph.

### My toy project

 Do you remember my post about [meaningful language]({% post_url 2016-09-26-a-meaningful-language %})? I quoted Harnad (1990):

 $$ Horse\ \&\ Stripes = Zebra $$

 I am trying out different models which learn compositionality from data. One of these models is recurrent neural network called LSTM, which basically can be used as an encoder-decoder ([Cho et al. 2014](https://arxiv.org/abs/1406.1078)) who takes a sequence of inputs and produce outputs. In this example, sequence of `["horse", "stripes"]` is input and `["zebra"]` should be the output. We encode each word with a vector to be able to do our magical mathematical operations, I will talk about word embeddings in future but lets jump to my toy example.

 What I will test here in my toy experiment is much simpler than my zebra example. The compositionality is much more clear in simple math operations such as adding two natural numbers $$ a + b = c $$:

$$ 1 + 2 = 3 $$

In my experiment, I will generate random examples of add operation: A sequence of words and their expected equivalence (e.g. `["1", "+", "2", "</eos>"]` as input and `["3"]` as target). I treated each operand ("1", "2", ...) and operator (e.g. "+") as symbols with no value. Basically, I used a simple [one-hot encoding](https://en.wikipedia.org/wiki/One-hot) to vectorize them. This simple code using [preprocessing of Scikit-learn](http://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.OneHotEncoder.html) would do this for me:

{% highlight python %}
from sklearn.preprocessing import OneHotEncoder

input_vocab = ["", "0","1", "2", "+", "="]
input_vocab_ids = [[0], [1], [2], [3], [4], [5]]
output_vocab = ["0", "1","2", "3", "4"]
output_vocab_ids = [[0], [1], [2], [3], [4]]

MAX_VOCABULARY_SIZE = 10
input_encoder = OneHotEncoder(n_values=MAX_VOCABULARY_SIZE)
input_encoder.fit(input_vocab_ids)

output_encoder = OneHotEncoder(n_values=MAX_VOCABULARY_SIZE)
output_encoder.fit(output_vocab_ids)
{% endhighlight %}

You can test it by printing the vector representation of "1" in this example:

{% highlight python %}
print("1 =>", input_encoder.transform([input_vocab_ids[input_vocab.index("1")]]).toarray())
# 1 => [[ 0.  0.  1.  0.  0.  0.  0.  0.  0.  0.]]
{% endhighlight %}

The LSTM model needs its own blog post I will explain that later. Story short, with a larger set of vocabulary the LSTM training process will take time but it converges to a state which can take string inputs and give you equivalence string outputs.

This model is naively simple and definitely doesn't learn meaning of number and addition as humans do. A simple test on unseen words shows that how weak is our design. But the interesting aspect is the ability to learn ordered composition from examples. using this model. I performed this training with numbers between 0 and 49. I over fitted the model with 97% accuracy on training set. The error rate still matches the rate of unseen combination (trained with 33% of all possible combinations and test on rest of them produces 99% error on unseen examples). But its performance on unseen examples is not very bad with different perspective. Consider this, all of these results are string, and my error rate is based on being correct agains incorrect. For example, an unseen combination "33 + 44 =" my system produces "78". But if you consider numeric values of these strings the variance of errors on unseen combinations are not very high (I don't have the results yet). The most important errors are in unseen (out of range) words.

Basically, in this naive model we didn't learn addition as an operation. All words in this model carry the meaning of addition because our corpus implies such relation between words. Even if I remove the sign "+" the result of "1 2" is "3". In my future tests I will try to break this simple composition and make my sample data more complicated.

I would like to comeback to my hidden claim that one-hot vector representations are just treating words as symbols with no values. Clearly, in my model I assigned list of numbers to each word. This is why I call it symbolic, I don't assign any numbers to any words by using extra knowledge than these words are different from each other, even though they can have same values (e.g. "" and "0" can have same value `0`). I even treated numbers and operators similarly. I could use my knowledge about the corpus and make a better loss function based on regression instead of cross-entropy but again I didn't because I simply didn't want use any math to learn math. But isn't it exciting that this naive model can say: "33 + 44 = 78"? It is wrong but it's not that bad :D.
