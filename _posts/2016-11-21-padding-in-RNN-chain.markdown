---
layout: post
title:  Padding in RNN chain
date:   2016-11-21 01:59:59 +0200
categories: blog
comments: true
---

I confess that I didn't share all of my codes on last post. Well, I was still playing with my new toy. It turns out that my implementation has some flaws. I made some changes but I would like to address a few tricks that I learned in this process.

First, padding matters. It's a bit strange. Maybe it's because I am a noob on TensorFlow, the famous language model RNN chain in TensorFlow does not have dynamic length. I had a fixed length for my RNN which I was filling the missing input gaps with paddings. But my padding technique wasn't quit correct. I had a one-hot vector for padding, which in fact was helping the model where it shouldn't. It was creating an illusion of high accuracy when my model was learning to predict padding vector. Now, in my next versions I am using zero vector (both on input paddings and output paddings). Ok, let's see how it effects the training in this post. Before that, I want to address the second missing part of the previous post: errors and validation.

So, this week I spend time on fixing it and trying to train a model but it turns out that it was overfitting in very wrong direction. I added the validation part (missing in previous post) in order to observe how and when the overfitting was happening. It turns out that my model only can tell difference between blue, green, purple and brown and it is very bad at doing other stuff. I will fix this problem soon by following up the original paper Monroe et al. 2016 using dropout, relu and there stuff in the paper. Let's back to the padding problem.

Well, maybe my padding technique is not wrong, but I would like to argue that contributing to my overfitting issue. Let just me remind what was the output the network for each sequence: a probability mass function for next word, our happy random variable :). In my implementation, I encoded paddings as a word! which means I had probabilities for it:

\begin{equation}
P(x_{t+1} = i|y_t) = softmax(y_t)(i)
\end{equation}

As I said, it might be cool for some tasks. I even had a faster training with these paddings. Well well, faster training sometimes is actually is equal to faster overfitting. Ok, let's check some math! Why was it faster? Because of my loss function:

Predicting i-th word in vocabulary with one-hot encoding, or its dirac distribution $$\delta_i$$:

\begin{equation}
\frac{\partial loss}{\partial \theta} = \frac{\partial H(\delta_i, softmax(y_t))}{\partial \theta} = \frac{\partial \sum_j{\delta_i(j)\times log(softmax(y_t)(j))}}{\partial \theta}  = \frac{\partial log(softmax(y_t)(i))}{\partial \theta}\
\end{equation}

While using zero vector:
\begin{equation}
\frac{H(\vec{0}, softmax(y_t))}{\partial \theta} = \frac{\partial \sum_j{0 \times log(softmax(y_t))}}{\partial \theta} = 0\
\end{equation}

Zero loss, zero gradient and zero learning! Zero is not even a valid distribution. We are correctly skipping the learning because there is nothing there to learn! So, previously:

{% highlight text %}
input: '<s> blue - -'
output: 'blue - - </s>'
{% endhighlight %}

Now, this is an example of my zero padding:

{% highlight text %}
input: '<s> blue - -'
output: 'blue </s> - -'
{% endhighlight %}
