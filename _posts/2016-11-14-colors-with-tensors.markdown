---
layout: post
title:  Colors with tensors
date:   2016-11-14 23:59:59 +0200
categories: blog
comments: true
---

Finally, my implementation of color descriptor in TensorFlow!

Two weeks ago I had post about color meaning. Today I want to go through an implementation of color descriptor. Basically, the resulted machinery will take a color (in HSV form) and produces a suitable description. I am basically trying to reimplement [this work](https://arxiv.org/abs/1606.03821).

These descriptions are learned from the Munroe's color survey preprocessed in [RUGSTK](http://mcmahan.io/lux/) (with very little modification which you can see here)

WARNING! You will see lots of preprocessing here! :D

## Preprocessing

After downloading the dataset (comes with LUX code), place it in your working folder. Then, you can start your code by adding the package path in python paths:

{% highlight python %}
import sys
sys.path.append('./rugstk_v1')
sys.path.append('./rugstk_v1/core')
sys.path.append('./rugstk_v1/data')
sys.path.append('./rugstk_v1/data/munroecorpus')

from rugstk_v1.data import munroecorpus
{% endhighlight %}

The first step is to create our dictionary of vocabularies in the corpus. In order to get all possible descriptions in LUX, we extract all keys from a training handle. Keys are the complete description, or textual part of the corpus.
In next step, we just need to parse them into chunks of words.

{% highlight python %}
training_hndl = munroecorpus.get_training_handles()
original_keys = list(training_hndl[0].keys())
# convert '-' to ' ' in order to get parts of words such as "greenish-blue".
color_phrases = [phrase.replace('-', ' ') for phrase in original_keys]
voc = set(word for phrase in color_phrases for word in phrase.split())
{% endhighlight %}

Now, that our vocabulary is ready, we can create an encoder which makes vector representation of these words.
The easiest way to encode vocabulary is one-hot encoding. Basically, the assumption of one-hot representations is that no word in vocabulary shares any feature with other words.
In other word, we have a feature space in size of the vocabulary, represents each symbol with a binary vector with `1` in its corresponding dimension and `0` in others, {0: absent, 1: present}.

$$
f_i :: \left[\begin{array}
{c}
0 \\
0 \\
... \\
1 \\
... \\
0 \\
0
\end{array}\right]\begin{array}
{l}
1 \\
2 \\
... \\
i \\
... \\
d-1 \\
d
\end{array} \forall\ i \in \{1, ..., d\}\ (index\ of\ features\ /\ words)
$$

It can also be interpreted as a vector representation for discrete probability distribution.

$$
P(w_i\ |\ w_i) = \left[\begin{array}
{c}
0 \\
0 \\
... \\
1 \\
... \\
0 \\
0
\end{array}\right]\begin{array}
{l}
1 \\
2 \\
... \\
i \\
... \\
d-1 \\
d
\end{array} \forall\ w_i \in VOC\ (vocabulary)\ \forall\ i \in \{1, ..., d\}\ (index\ of\ words)
$$

The number of dimensions $$d$$ is equal to the size of our vocabulary $$d=\|VOC\|$$. We can also represent the the vocabulary with an identity matrix.

$$
I_{d \times d} = \left[\begin{array}
{cccccc}
1 & 0 & ... & 0 & ... & 0 \\
0 & 1 & ... & 0 & ... & 0 \\
... & ... & ... & ... & ... & ... \\
0 & 0 & ... & 1 &... & 0  \\
... & ... & ... & ... & ... & ... \\
0 & 0 & ... & 0 & ... & 1
\end{array}\right]\begin{array}
{l}
1 \\
2 \\
... \\
i \\
... \\
d
\end{array}
$$

I am using the one-hot encoder in scikit-learn, however, you can invent your own version:

{% highlight python %}
import numpy as np
from sklearn.preprocessing import OneHotEncoder
# a ridiculous size of vocabulary for future expansion
MAX_VOCABULARY_SIZE = 1000

# one-hot encoder put probability 1.0 for the seen type and zero for other types.
one_hot_encoder = OneHotEncoder(n_values=MAX_VOCABULARY_SIZE)
one_hot_encoder.fit(list([i] for i in range(len(voc))))
{% endhighlight %}

This function might be useful later, in order to convert a string into a sequence of vector representations:

{% highlight python %}
# feature extractor:
def feature_extractor(sentence):
    for word in sentence.split():
        if word in voc:
            yield [voc.index(word)]
        else:
            yield [voc.index('<unknown>')]

# example usage:
example_vectorized = one_hot_encoder.transform(list(feature_extractor("ugly dark blue green"))).toarray()
print("P(X|ugly), P(X|dark), P(X|blue), P(X|green) = ")
print(example_vectorized)
{% endhighlight %}

{% highlight text %}
P(X|ugly), P(X|dark), P(X|blue), P(X|green) =
[[ 0.  0.  0. ...,  0.  0.  0.]
 [ 0.  0.  0. ...,  0.  0.  0.]
 [ 0.  0.  0. ...,  0.  0.  0.]
 [ 0.  0.  0. ...,  0.  0.  0.]]
{% endhighlight %}

This is the inverse of the function, which we will need in future:

{% highlight python %}
# the reverse process:
def features_to_types(vectors):
    for vector in vectors:
        yield(voc_prep[np.argmax(vector)])

print("The reversed process:\nargmax_x{P(x|word)} = word")
print(" ".join([word for word in features_to_types(example_vectorized)]))
{% endhighlight %}

{% highlight text %}
The reversed process:
argmax_x{P(x|word)} = word
ugly dark blue green
{% endhighlight %}

In next section, we will read features and prepare them for our neural nets.

## Prepare the dataset

The challenge in this stage is to understand the input data, output data and how you want read it while you don't blowdown your memory.

Ok, here it is what I want to do: I want to create two arrays in parallel, `X_train` as input, and `Y_train` as output.
They consist of samples from datapoint, each sample has a sequence of description words, the input starts with start tag '<s>' and output ends with end tag '</s>':

For example:

{% highlight text %}
input: '<s> ugly dark blue'
output: 'ugly dark blue </s>'

input: '<s> dark blue -'
output: 'dark blue - </s>'
{% endhighlight %}

The only thing we need to do is to convert them in to vectors instead of taking them as strings.
In addition to this, each word-vector in input, according to the decoder model, needs to be concatenated with color feature vector.

This is almost all of the story. I will talk about the test and development in future.

One missing piece is reading from memory! the training input is a 3-dimensional color code and a sequence of word-vectors are in size of the vocabulary. The length of sequence is fixed and we just pad it with zeros when the description is shorter than maximum length.
Let's do a simple math:

(3-d color = float x 3) x (max_sequence_length = 3) x (vocabulary size = float x 337) x (number of samples = 1.5 million) =>
16*3*3*16*337*1500000 / (1024*1024*1024) = 1 tera bytes

Conclusion: we need a way to read mini-batches from file in a size which fits our memory.

{% highlight python %}
COLOR_FEATURE_SIZE = 3 # h,s,v vector

MAX_PHRASE_LEN = 4
PADDING_VEC = one_hot_encoder.transform([[voc_prep.index("")]]).toarray()
BEGIN_VEC = one_hot_encoder.transform(list(feature_extractor("<s>"))).toarray()
END_VEC = one_hot_encoder.transform(list(feature_extractor("</s>"))).toarray()

# we already have the preprocessed data handler:
#training_hndl = munroecorpus.get_training_handles()

def color_to_features(color_vector):
    # no additional process?
    h,s,v = color_vector
    return [
      h,
      s,
      v,
    ]

# read all the dataset from file and load it into RAM:
def read_in_batch(batch_size, randomize=True):
    X_train = []
    Y_train = []

    # randomize the batches for each epoch
    phrases = list(enumerate(color_phrases))
    if randomize:
        np.random.shuffle(phrases)

    for phrase_id, phrase in phrases:
        phrase_vectors = one_hot_encoder.transform(list(feature_extractor(phrase))).toarray()

        # being, end and padding
        padding_number = MAX_PHRASE_LEN - phrase_vectors.shape[0]
        phrase_vectors = np.concatenate((
                BEGIN_VEC,
                phrase_vectors,
                np.repeat(PADDING_VEC, padding_number, axis=0),
                END_VEC
            ), axis=0)

        color_key = original_keys[phrase_id]
        # datapoints:
        # dim = 0,1,2 for h,s,v
        with open(munroecorpus.get_training_filename(color_key, dim=0)) as h_file, open(munroecorpus.get_training_filename(color_key, dim=1)) as s_file, open(munroecorpus.get_training_filename(color_key, dim=2)) as v_file:
            for h_line, s_line, v_line in zip(h_file, s_file, v_file):
                # text to float:
                h_data = float(h_line.replace('\n',''))
                s_data = float(s_line.replace('\n',''))
                v_data = float(v_line.replace('\n',''))
                # repeat the vector:
                color_feature_vector = color_to_features([h_data, s_data, v_data])
                color_vectors = np.array([color_feature_vector] * (MAX_PHRASE_LEN+1))

                # create the sample input (encoded color+word)
                X_train.append(np.concatenate((color_vectors, phrase_vectors[:-1]),axis=1))
                Y_train.append(phrase_vectors[1:])

                # debug:
                # print(color_key, color_feature_vector)

                if len(X_train) == batch_size:
                    yield X_train, Y_train
                    X_train = []
                    Y_train = []

    yield X_train, Y_train

# debug
for d in read_in_batch(5, len_step=2):
    #print(d)
    break
{% endhighlight %}

## TensorFlow

The simplest line of code: import the library.

{% highlight text %}
import tensorflow as tf
from datetime import datetime
{% endhighlight %}

As I mentioned in one of my previous posts, you can see TensorFlow in two phases of programming.
First, you create a graph of tensors and their flow in network with operations. Then, you run a session which there you can feed data.

### Graph of tensors and operations

Most things that you write in TensorFlow like `tf.something` are nodes of a graph.  I don't know the best practice now, but I know that if things are simple and readable means it's good to go. So, first trick is to always reset the graph before writing anything in graph. This makes sure that you are not problem free in interactive mode:

{% highlight text %}
tf.reset_default_graph()
{% endhighlight %}

Then we can start with creating placeholders for data which will be fed in the graph (input and output):

{% highlight text %}
# batch size is arbitrary= None
X = tf.placeholder(tf.float32, shape=(None, MAX_PHRASE_LEN+1, COLOR_FEATURE_SIZE+MAX_VOCABULARY_SIZE))
Y = tf.placeholder(tf.float32, shape=(None, MAX_PHRASE_LEN+1, MAX_VOCABULARY_SIZE))
{% endhighlight %}

Let me go though the details. placeholder, creates an input feed for the graph. It defines the type and shape of the tensor here. Tensors are high dimensional vectors and matrixes. In this code, I defined a placeholder called `X` and `Y` with type float32, and a 3D shape dimensions. Notice that the matrix is not 3D, the shape of dimension of the matrix comes with 3 numbers, The first number is not defined, which means arbitrary size, and other two size indicates the two other sizes of dimensions.

The first number in shape means that we can send any number of batch samples. The second number defines a fixed length of sequence and third number is the size of the feature-vector in each time-step of the sequence. (the input vectors has both color features and word features)

Now we need to define a chain of RNN and plug the sequence input and sequence out put in it. The chain of LSTM must be made out of one cell (in this setup dimensions of output vectors from each cell will be 20).

{% highlight text %}
hidden_dimension = 20
lstm_cell = tf.nn.rnn_cell.LSTMCell(hidden_dimension, state_is_tuple=True)
output, state = tf.nn.dynamic_rnn(lstm_cell, X, dtype=tf.float32)
{% endhighlight %}

Where the `tf.nn.rnn_cell.LSTMCell` instantiates the cell, and `tf.nn.dynamic_rnn` creates a chain of these cells `lstm_cell` in size of the input tensors `X`.

All these outputs from chain, should go though a fully connected network which translates outputs into logits of predictable categories. A fully connected network is just a weight matrix and a bias vector. We just need to be careful about its size, type:

{% highlight text %}
# weight and bias initialization
weight = tf.Variable(tf.truncated_normal([hidden_dimension, int(Y.get_shape()[2])]))
bias = tf.Variable(tf.constant(0.1, shape=[Y.get_shape()[2]]))
{% endhighlight %}

Maybe now we are in the complicated part of the process. Because we need to take the output of each cell and multiply it with weights then add the bias. I want to remind you that tensors are not like numpy arrays, they don't any data in them. They are like concept which we can create an operations to process them. In order to separately get each output out of chain of cell, we cannot call use their index like arrays. Instead we have to apply an operation like `tf.gather` which chose a tensor in a list of tensors. Since the shape of `output` shows the batches first, we cannot use gather, unless we transpose the outputs on and reshape it to a list of sequence out puts instead of list of batches. This easy:

{% highlight text %}
output = tf.transpose(output, [1, 0, 2])
{% endhighlight %}

Now logit predictions for each time space output can make use `tf.gather`:

{% highlight text %}
# logits (predictions),
predictions = [
    tf.add(tf.matmul(tf.gather(output, i), weight), bias)
    for i in range(int(output.get_shape()[0]))
]
{% endhighlight %}

Let me remind you, that these predictions are stepwise outputs which represent probability function of possible outcome of the chain conditioned with previous inputs.

Now it's time to define the loss function and the trainer which will update all parameters based on the loss. So, again, each prediction logits with a softmax function will become a probability distribution predicting the class of the target output. During the training where we have the correct answer, we want to improve this distribution toward the right answer. Basically, we want the this distribution be more similar to a distribution which only the correct category wins the prediction. Based on this intuition, the cross-entropy distance is our loss function:

{% highlight text %}
losses = tf.convert_to_tensor([
    tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(predictions[i], tf.gather(tf.transpose(Y, [1, 0, 2]), i)))
    for i in range(int(output.get_shape()[0]))
], dtype=tf.float32)

optimizer = tf.train.AdamOptimizer()
minimize = optimizer.minimize(losses)
{% endhighlight %}

With loss, we only need to choose a right stochastic gradient descent technique. In this case, I have AdamOptimizer. (The original paper has a different model)

And finally, I have another tensor which will represent the error rates after each prediction:

{% highlight text %}
errors = tf.reduce_mean([
    tf.reduce_mean(tf.cast(
        tf.not_equal(tf.argmax(tf.gather(tf.transpose(Y, [1, 0, 2]), i), 1), tf.argmax(predictions[i], 1)),
        tf.float32
    ))
    for i in range(int(output.get_shape()[0]))
])
{% endhighlight %}

Using argmax gives us the index of the correct answer from Y and we are comparing it with argmax in prediction vectors. This code basically will calculate the average of errors in each cell output.

## Running the session

Feeding the graph of tensors is the final stage. Before doing it, I want to remind that we need to save variables after training otherwise we have to retrain the model each time! (days and weeks of training)

{% highlight text %}
# saver for saving the model on a file when ever we ask for it.
saver = tf.train.Saver()
{% endhighlight %}

Let's initialize the variables before flowing the data in our graph:

{% highlight text %}
# initialize the variables according to their setup (e.g. random values)
init_op = tf.initialize_all_variables()

# start the session with
sess = tf.Session()
sess.run(init_op)
{% endhighlight %}

Now we can train it:

{% highlight text %}
batch_size = 34500
epoch_number = 12
for epoch in range(epoch_number):
    total_samples = 0
    epoch_error = 0
    minibatch_counter = 0
    for X_train, Y_train in read_in_batch(batch_size):
        minibatch_counter += 1
        real_sample_size = len(X_train)
        _ , err = sess.run([minimize, errors], {
            X: X_train,
            Y: Y_train,
        })

        print("minibatch-error: {:6.2f}%, minibatch_counter: {:2d}".format(100.* err, minibatch_counter))
        total_samples += real_sample_size
        epoch_error += err*real_sample_size

    now_time = datetime.now().strftime("%Y%m%d%H%M")
    saver.save(sess, "./tmp/%s-%s-model.ckpt" % (now_time,epoch))
    summary_writer = tf.train.SummaryWriter('./tmp/%s-%s-model/log'%(now_time,epoch), sess.graph)
    print("epoch: {:1d}, epoch_error: {:6.2f}%, total_samples: {:6d}, model: {}".format(epoch, 100.* epoch_error/total_samples, total_samples, "./tmp/%s-%s-model.ckpt" % (now_time,epoch)))
{% endhighlight %}

Lots of work, ha?

### The top 10 descriptions

In addition to the learning model. I wanted to have code to show how it works. Then I thought let's write a beam search which finds top something descriptions and their probability!

Here is the code, I will add more description later. Probably there are better implementations out there, this is just a quick respond.

{% highlight python %}
n_beam = 10
phrase_vectors_inits = [np.array(BEGIN_VEC)]
prev_choices = [(list(feature_extractor("<s>")),0)]
for i in range(MAX_PHRASE_LEN+1):
    curr_choices = []
    for j in range(len(prev_choices)):
        padding_number = MAX_PHRASE_LEN - phrase_vectors_inits[j].shape[0] + 1
        phrase_vectors = np.concatenate((
                phrase_vectors_inits[j],
                np.repeat(PADDING_VEC, padding_number, axis=0)
            ), axis=0)
        color_vectors = np.array([test_color] * (MAX_PHRASE_LEN+1))

        X_input = np.concatenate((color_vectors, phrase_vectors),axis=1)

        results = sess.run(predictions, {
            X: [X_input]
        })

        # top n_beam of this level:
        word_ids = np.argpartition(results[i][0], -n_beam)[-n_beam:]
        # new set of choices:
        # word_id and its logit
        curr_choices.append([
                ([word_id], softmax(results[i][0])[word_id])
                for word_id in word_ids
            ])

    new_paths = sorted([
        (prev_log_prob + np.log(new_prob), prev_word_ids+[new_word_id])
        for j, (prev_word_ids, prev_log_prob) in enumerate(prev_choices)
        for new_word_id, new_prob in curr_choices[j]
    ])

    prev_choices = [(word_ids, log_prob) for log_prob, word_ids in new_paths][-n_beam:]
    phrase_vectors_inits = [
        one_hot_encoder.transform(word_ids).toarray()
        for word_ids, log_prob in prev_choices
    ]

for word_ids, log_prob in prev_choices:
    print(np.exp(log_prob), [voc_prep[word_id[0]] for word_id in word_ids])

{% endhighlight %}
