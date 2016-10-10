---
layout: post
title:  Working on weekends
date:   2016-10-09 23:59:59 +0200
categories: blog
comments: true
---

I was playing with [TensorFlow](https://www.tensorflow.org) this weekend. But I didn't have my work laptop with me. Instead of setting up everything on my home computer I had SSH access to run iPython notebook remotely and continue the job on browser. I will share the setup here, maybe you find it useful :)

Ok, first let's see what am I talking about. I have a Mac laptop 'A' with my iPython projects. I have a [Hamachi VPN](https://en.wikipedia.org/wiki/LogMeIn_Hamachi) connection to 'A' (it's up to you! you can find your own solution) and [remote login access](https://support.apple.com/kb/PH18726) to it. I have physical access to my computer 'B' which I have a typical SSH client and a browser (comes in all Mac or most Linux machines). Here is the idea, I will remotely connect to my device on 'A' run the `jupyter notebook` and open browser on 'B' to work on my projects. My imaginary username on 'A' is `work-user` and on 'B' is `home-user`. The IP address of 'A' is `192.168.1.111` in this example. The terminal on these machines show usernames as initials on shell such as `home-user$` on 'B'.

(You cannot copy paste any of the codes as a ready to use commands, you have to move forward line by line, understand them ,and edit them based on your setup)
First, we want to remotely open a shell access on 'A'. In order to run a stable iPython on 'A', I would like to create a `screen -a` shell terminal which is not dependent to the ssh connection and it will be possible to reconnect and see that terminal in future:

{% highlight shell %}
home-user$ ssh -l work-user 192.168.111
Password:
work-user$ screen -a
bash-3.2$
{% endhighlight %}

After you saw the new terminal line, you are ready to run the iPython in our project folder. It doesn't necessarily show `bash-3.2$`. Running the iPython on your project folder is optional in my example I have this path for my iPython files: `/the-project-path/`.

{% highlight shell %}
bash-3.2$ cd /the-project-path/
bash-3.2$ jupyter notebook
[I 12:16:13.791 NotebookApp] Serving notebooks from local directory: /the-project-path/
[I 12:16:13.791 NotebookApp] 0 active kernels
[I 12:16:13.792 NotebookApp] The Jupyter Notebook is running at: http://localhost:8888/
[I 12:16:13.792 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
{% endhighlight %}

Now that we have running server on localhost on 'A', Back on 'B' we want to be able to open it on browser. But first we have to forward this connection to localhost of 'B'. I chose to forward it on similar port (`8888`) on 'B'. So, open a new terminal on 'B':

{% highlight shell %}
home-user$ ssh -l work-user -L 8888:127.0.0.1:8888 192.168.111
Password:
work-user$
{% endhighlight %}

It's ready to go! Open the browser and go to `http://127.0.0.1:8888`. I will write about [TensorFlow](https://www.tensorflow.org) experience maybe next week!
