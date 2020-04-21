# My Quarantine Playlist

When Philadelphia first went into self-quarantine last month, realizing that I was going to be working from
home for the foreseeable future, I went to building a playlist that I could listen to for a "while" without getting bored
of.

Such a playlist would need to be fairly large, but I didn't want to spend
too much time curating it so I put together the following rules to determine which
songs should be included.

1. Add all albums released between 2000 and 2010 that had at least one song I listened to
    during that time period. This accumulated about 48 hours of music.
2. Each day open up Spotify and restart the playlist on shuffle, removing songs I don't like
    as they are played.

After a few weeks my wife was surprised to see that I was still removing songs
from the playlist and that there were yet songs which hadn't been played. Since
this is a probabilistic experiment, this raised some interesting questions.

For example, without listening to the entire playlist, can I estimate

1. How many songs I've listened to so far?
1. How many songs are left that I'm going to remove?
2. When I've removed all songs from the playlist?

# Where are we?

[Bagging](https://en.wikipedia.org/wiki/Bootstrap_aggregating),
the generalization of Random Forests and which stands for "bootsrap aggregating",
is not a bad place to start when thinking about answering the first question above.
In bagging, one samples with replacement from a given data set to create $m$ new data sets,
trains a model on each of these new trainings sets and then aggregates the results.

While model training is not relevant here the process of sampling with replacement is. If you read
the Wikipedia page on bagging you'll come across the fact that each of the $m$ data sets are made
up of approximately 63% unique examples. (The actual number is $1-1/e$, if you're interested in
the calculation see [this stackexchange post](https://math.stackexchange.com/questions/32800/probability-distribution-of-coverage-of-a-set-after-x-independently-randomly?noredirect=1&lq=1)). This calculation assumes that each of the $m$ samples
has the same number of examples as the original data set, however in general the number of unique 
samples is

$$
n\left(1 - \frac{1}{N}\right)^{n}
$$

where $n$ is the number of samples and $N$ is the population size we are sampling from
(number of examples in the original data set).

In my case, I don't know the exact values of $n$ or $N$ but 800 and 960 are educated guesses.
This gives us $\approx 347$ unique songs listened to, or said another way, out of the roughly
800 songs I've listened to so far, 453 are repeats.

# How much longer till we get there?

Now that we have an idea of how much of the playlist I've listened to so far,
let's consider estimating how many songs remain in the playlist that I'm ultimately
going to remove after they play - for convenience I'll refer to these songs as
*bad* going forward, with emphasis to avoid suggesting that my personal taste
of music reflects any sort of objective quality.

A standard approach is to begin with a sample of the playlist. That is, let
it play on shuffle and keep track of how many songs are played in total ($n$)
and how many were *bad* ($s$).

While writing this post I  did exactly that, listening to a total of 21 songs,
6 of which were *bad*.

Now that we have some data,
$D=(n=21, s=6)$, and a parameter we would like to estimate, namely the proportion of 
*bad* songs in the playlist we have all of the main
ingredients to calculate $P(p\vert D)$ using Bayes rule (supposing we don't
care to treat certain values of $p$ as more likely than others).

In particular the probability that $p=\tilde{p}$ for $\tilde{p}\in[0,1]$
works out to

$$
P(p=\tilde{p}\vert n,s)=\frac{(n+1)!}{s!(n-s)!}\tilde{p}^{s}(1-\tilde{p})^{n-s}
$$

from which we can obtain obtain the following posterior probability for $p$.

![](how-much-longer.png)

This shows the mode is centered at the maximum likelihood estimate of $6/21\approx .28$, but
also suggests that values in the neighborhood of $[.2, .5]$ are also likely estimates for the proportion
of *bad* songs in the playlist.

For details on deriving the equation above see [here](https://en.wikipedia.org/wiki/Rule_of_succession#Mathematical_details).

# Are we there yet?

Finally "how will I know that all of the *bad* songs have been remove from the playlist?"

If we begin with sampling from the playlist once again we will find ourselves in a similar
situation as before, however with two key differences.

First, let's work with the principle that it would be premature to begin considering a criterion
for testing whether all *bad* songs have been removed from the playlist
before being reasonably sure that few, if any, *bad* songs remain in the first place.
Operating under this principle, we wouldn't want our criterion to be 
unduly "influenced" by large, irrelevant values of $p$.

In other words, unlike before, we want to explicitly treat some values as more likely than others,
which is of course flirting with the idea of a prior. 

We'll use the same underlying calculations to estimate $p$ as before, but this 
time placing a non-uniform prior on the probability of the value of $p$. In particular we'll use
$p\sim\text{Beta}(1, 20)$, which is a suitable choice for two reasons. First, the Beta distribution
has a special relationship with the binomial distribution which makes
[simplifies calculating the posterior](https://en.wikipedia.org/wiki/Binomial_distribution#Estimation_of_parameters).
Second, and more importantly, the prior easily expresses the belief that few, if any, *bad*
songs remain in the playlist, as seen in the following plot.

![](./beta-pdf.png)

The second important distinction to make from how we treated the sampling process above
is that we are now considering a binomial trial where the number of successes (when a *bad*
song plays) is *always* 0.
This is because the moment a *bad* song plays it is immediately known that $p\not=0$.
Thus the only value in our calculation that changes from one sample to the next is $n$,
and we can do the calculations for $n=1\ldots$ *before* sampling.

Given $n$ trials with 0 songs found to be deleted we can calculate the posterior of $p$
as

$$
P(p|n,\alpha=1,\beta=20)=\frac{1}{21 + n}
$$

The following plot shows how our estimate of $p$ is updated after $n$ consecutive "not *bad*"
songs play.

![](./beta.png)

Of course, we can never know for sure that $p=0$ without listening to the entire playlist.
However, the plot shows that with a little algebra we can easily develop a criteria that
allows us to be reasonably sure that $p$ is very close to 0 without listening to anywhere
near the entire playlist. For example if 80 songs play, without any one of them being *bad*
we can be reasonably sure that $p<0.01$.

# Lessons learned

Obviously this is a toy problem, but are there any useful take aways? I think so.

In my opinion, the main lesson to be learned here that having a
familiarity with "variety" of well known problems can be immensely valuable
when approaching something new if you can recognize which ones are relevant.

For example, while the calculations borrowed from bagging above are not perfectly
suited to the actual problem we considered here it's nevertheless a useful starting point.
In bagging one takes $n$ samples *with* replacement $m$ times.
However, here the setup is to take $n^{\prime}<n$ samples *without*
replacement $m$ times. Moreover, because *bad* songs are being remove in real time
$n$ is changing all the time. These are obviously different sampling procedures, where the
latter is more difficult to solve. Nevertheless, as $m$ grows, our experiment will begin
to look more like sampling with replacement. Thus, by identifying a similar, but simpler,
problem we can quickly arrive at an approximation perhaps buying and serve as a sanity
check while working on the more difficult problem.

For another example, consider the calculations we used to estimate the proportion of songs
remaining in the playlist that I will remove. At first glance this is a straightforward application
of Bayes theorem that doesn't require any further context. Before simplifying
terms, the expression is

$$
P(p\vert n,s)=\frac{P(n,s\vert p)P(p)}{P(D)}
$$

While the numerator is a simple maximum likelihood estimate calculation and a flat prior
which are easy to calculate, the denominator requires an integral that is much more difficult.

$$
P(D)=\int_{0}^{1}{q^{s}(1-q^{n-s}) \ \text{d}q}
$$

In fact, this reminds us that often $P(D)$ cannot be calculated analytically. Fortunately the problem we are
trying to solve here is the well-known [Rule of Succession](https://en.wikipedia.org/wiki/Rule_of_succession)
whose wiki page reminds us

$$
\int_{0}^{1}{q^{s}(1-q^{n-s}) \ \text{d}q}=\frac{s!(n-s)!}{(n+1)!}
$$

Of course, there's always Wolfram Alpha for those, like myself, whose calculus skills are a bit
rusty, but recognizing that this is well studied problem gives us
access to a bit more context and might help point us to other useful resources.
