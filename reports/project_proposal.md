# Social Clustering and Individualism
**Jeremy Ryan**

### Abstract
Within a population of humans, there tends to be a balance
of diversity and clustering of opinion. That is, for any 
continuous stance (say, degree of support for speed limit 
enforcement) there tend to be social clusters with some
amount of consensus within them and some amount of variance
between each cluster.

Based on prior work in this area, I plan to implement a Python
model to recreate this clustering behavior, and present results
using MatPlotLib.

### The Project
In a paper titled [Individualization as Driving Force of 
Clustering Phenomena in Humans](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1000959),
authors Mäs, Flache, and Helbing develop an agent-based model
to serve as a potential explanation for this behavior by
balancing a tendency for conformity with a tendency for
individualism. Their simulation was able to successfully
create a system where agents would form clusters of opinion
that changed dynamically over time, without inevitably stabilizing
into a single monoculture or dissolving into chaos.

In the words of the authors:

> We have developed a formal theory of social influence that, 
> besides anomie and monoculture, shows a third, pluralistic phase 
> characterized by opinion clustering. It occurs, when all 
> individuals interact with each other and noise prevents the 
> convergence to a single opinion, despite homophily.

This is offered as a counterargument to claims that globalization
and widespread use of the Internet will lead to monoculture.

### Experiments and Extensions
For this project, I hope to
- **Implement** the agent-based model detailed in the Mäs paper
- **Reproduce** the results observed in the paper, including a
plot of agent opinion over time that shows distinct clusters
- **Expand** the model to include spacial relationships between
agents, either through a graph representation or a model where
agents move freely within a 2D space

With my extension(s), I hope to gain more insight into opinion
clustering with spatially-separated agents, where each agent is
influenced more strongly by agents that are nearby than those that
are far away. Tuning the strength of this effect may give
results that parallel social effects as the Internet closes the 
"distance" separating existing groups and cultures.

