# Optimization-of-PageRank-Algorithm-using-Parallelization-Methods
<div align="center">
  <img src="https://raw.githubusercontent.com/lionell/pagerank/master/docs/img/googlepagerank.jpg" />
</div>

Implementation of [PageRank](https://en.wikipedia.org/wiki/PageRank) algorithm, using
different parallelization approaches. For now there are three of them available:
Serial(no parallelization), OpenMP(shared memory), MPI(communication via network).
We are going to benchmark them on different datasets.

## PageRank

PageRank is an algorithm used by Google Search to rank websites in their search engine results.
PageRank was named after Larry Page, one of the founders of Google.
PageRank is a way of measuring the importance of website pages. According to Google:

> PageRank works by counting the number and quality of links to a page to determine
> a rough estimate of how important the website is. The underlying assumption is that
> more important websites are likely to receive more links from other websites.

It is not the only algorithm used by Google to order search engine results,
but it is the first algorithm that was used by the company, and it is the best-known.

## Algorithm

There are different approaches of PageRank evaluation. They mostly differ in underlying
data structure(there is also MapReduce version). Method we used is great for distributed
computing, because we can minimize data transfered via network, and gain benefit from
shared memory.

For now we are going to call graph vertecies - pages. For each page we store list of
in-links. Eg. [3, 0, 1] means that our page has in-links from pages 0, 1 and 3.
Also we need to store amount of out-links for every page. Having these let's take a
look how we can compute PageRank efficiently.

Basically all approaches to computed PageRank are **iterative**. Let's divide one iteration
into three parts: PR from pages, PR from dangling pages and PR from random jumps.

### Pages

To compute PR from pages, we need to spread old PR(from previous iteration) across all the
pages linked with current. For example, if there are links from page 1 to pages 0, 2, 4, then
each of these pages will receive additional `PR[1] / 3` PageRank.

As you can see, we can use our data representation to compute this part. To compute PR of
some particular page, we can sum PRs of all in-pages divided by out link count for each page.

```
PR1[i] = PR[in_links[0]] / out_link_cnts[in_links[0]] + ... + PR[in_links[k]] / out_link_cnts[in_links[k]]
```

### Dangling pages

Dangling pages - are pages with no out-links. It's obvious that one will leave this page at some point
of time, and go to some random page. That's why we need to count these pages as they have out-links to
every single page in graph. But this will **significantly** increase size of in-links for every page.

It's pretty obvious that addtitional PR from dangling pages, is the same for all pages. So we can evaluate
it only once, and then add it to every page.

```
PR2[i] = PR1[i] + Dangling_pages_PR / page_cnt
```

### Damping factor and random jumps

In original page from Larry Page and Sergey Brin, they use some factor called damping factor to model
situation when user just stop web-surfing and go to random page. We will call this situation random jump.
To deal with this, we need damping factor(near 0.85), to say that with probability 0.85 user will continue
web-surfing, otherwise go to random page with probability of 0.15.

```
PR[i] = PR2[i] * damping_factor + (1 - damping_factor) / page_cnt

