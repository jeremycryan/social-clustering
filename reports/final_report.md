# Social Clustering and Individualism
**Jeremy Ryan**

### Abstract
Within a population of humans, there tends to be a balance
of diversity and clustering of opinion — that is, for any 
continuous stance (say, degree of support for speed limit 
enforcement) there tend to be social clusters with some
amount of consensus within and some amount of variance
between each cluster. This phenomenon is called pluralism.

This begs the question: on what conditions are pluralistic
behaviors hinged? Based on prior work in this area, have 
implemented a Python model to reproduce pluralistic behavior,
and expanded the model to represent more nuanced agent states. 

### Prior Work
Pluralism, though common in social systems, may seem paradoxical 
at first. Without some tendency to agree with the rest of the 
population, opinions would be distributed randomly; but without 
some tendency to diverge from consensus, all members of the population 
would share identical opinions.

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

### My Implementation of the Mäs Model
You can find [a Jupyter notebook containing my implementation](https://github.com/jeremycryan/social-clustering/blob/master/code/project_notebook.ipynb)
on Github.

#### Model Overview
In the Mäs model, a number of agents exist in a population. These agents
have an opinion, which can be any continuous value between -250 and 250.
Agents do not maintain any state other than their opinion value, and
have knowledge of the opinions of all other agents in the population.

Each time step, some sample of agents each observe the state of the
population, then calculate some ```Δo``` by which to change their
opinion before the next time step.

#### Conformity in Agents
The value of ```Δo``` for a particular agent is a sum of two terms, the
first of which creates a tendency to share the opinion of other agents.

First, a "social influence" value is calculated for each other agent.

```    
def social_influence(self, other):
    """ Calculate the degree to which another Agent influences
        this one; increases with more similar opinion
    """
    exponent = -abs(self.opinion - other.opinion)/self.A
    return math.exp(exponent)
```

This value represents the degree to which any agent has influence on the
active one, under the assumption that agents are more easily swayed by
other agents with similar beliefs. Notice that ```self.A```, a tunable
constant, acts as a way to scale social "distance" --- opinions that
differ by ```A``` will diminish social influence by a factor of e.

Next, this social influence value is used to determine the first term of
```Δo```:

```    
def total_social_influence(self):
    """ Returns a weighted sum of difference of opinion between other agents,
        with weights of each other agent's social influence, normalized
        to the total social influence of all other agents.
    """
    self.social_influences = {}
    for other in self.other_agents():
        self.social_influences[other] = self.social_influence(other)
    
    numerator = sum([(other.opinion - self.opinion) 
                     * self.social_influences[other] 
                     for other in self.other_agents()])
    
    denominator = sum([self.social_influences[other]
                      for other in self.other_agents()])
    
    return numerator/denominator
 ```
 
The first three lines of this method serve only to avoid redundant 
calculations of individual social influence values.

The numerator of this term is a weighted sum of all opinion
differences with other agents, weighted by that agent's social
influence on the actor. The denominator normalizes this term to the
total social influence of all agents.

#### Individualism in Agents
 
The second term in ```Δo``` creates a tendency for agents to deviate
from their neighbors.

```    
def gaussian_term(self):
    """ Returns a random value, based on parameters A and s. """
    
    std_dev = self.s * sum([self.social_influences[other]**self.A
                            for other in self.other_agents()])
    return random.gauss(0, std_dev)
```

This term introduces randomness into the model — the return term is
literally calling ```random.gauss``` — as a means to make opinions
deviate from monoculture. Interestingly, this term becomes more volatile
with more *homogenous* populations, since it relies on a sum of social
influence values (which you will recall are higher for like-minded
agents). Furthermore, ```self.s``` exists as a tunable constant to scale
the strength of this random deviation.

This is then simply summed with the first term:

```
def update(self):
    """ Updates the current agent's opinion based on the state of the population. """
    self.opinion += self.total_social_influence() + self.gaussian_term()
    self.apply_opinion_threshold()
    self.reset_stored_vars()
```

*Note that ```self.apply_opinion_threshold()``` limits the values of 
opinions to [-250, 250], and ```self.reset_stored_vars()``` clears
dictionaries of stored values between time steps.*

### Initial Model Validation

Using this model, Mäs et al were able to create the following 
visualization.

![image](https://github.com/jeremycryan/social-clustering/blob/master/images/clustering.png?raw=true)

In the figure above, which simulated 100 agents over 10,000 time
steps, agent opinion starts uniformly distributed in a narrow range
and then proceeds in what resembles a random walk. At points, the bulk
of agent density splits into two or more branches. Qualitatively, this
seems to support the claim that this model exhibits pluralistic behavior
with the right constant values; this figure used values of ```A = 2```
and ```s = 1.2```.

I was able to generate a qualitatively similar plot from my
implementation of the model.

![My figure](https://github.com/jeremycryan/social-clustering/blob/master/images/social_clustering_plot_1.png?raw=true)

This particular figure was generated with constant values of ```A = 2```
and ```s = 3.0```. I am unsure why I had to increase the value of 
```s``` for my simulation, because the models should be behaving
identically.

Regardless, my plot displays similar qualitative behavior that suggests
that pluralism is possible to produce in this model. Both my
visualization and the figure in the Mäs et al paper show clustering that
changes over time, reforms into different groups, and diverges again.

### Expanding to Two Dimensions
In reality, the opinions of individuals are much more nuanced than a
single scalar value. People hold a variety of different opinions, and
the extent to which individuals are similar is based on a combination of
each individual's several axes of opinion.

As such, I expanded the Mäs model in an effort to determine whether the
relatively simple behavior of agents could still form pluralistic
behavior when expanded to multiple dimensions. In this new model,
social influence is calculated using Euclidean distance between all
axes of opinion.

I tested this model in a 2D opinion space, and had limited success
reproducing pluralistic behavior. In general, agents grouped together
in a single cluster, which tended to have regions of higher and lower
agent density, and occasionally broke off into small clusters of ten or
fewer agents.

You can see an animated version of this plot over time in the Jupyter
notebook in this repository.

![Image of clustering](https://github.com/jeremycryan/social-clustering/blob/master/images/clustering2d2.png?raw=true)

*Pluralistic behavior is limited in the 2D model.*

In an effort to quantify this behavior, I used mean shift clustering to
count the number of agent clusters in the opinion space over time.
This algorithm counts clusters by simulating an array of points that
move toward areas of higher point density, eventually converging on
some number of maxima, then counting them.

Below is a plot of clustering over time.

![Clustering over time](https://github.com/jeremycryan/social-clustering/blob/master/images/number_of_clusters_over_time.png?raw=true)

The first point of interest on this plot is that the number of clusters
is changing over time, sometimes substantially. It has a minimum of
one cluster (as it contains at least one point at all times), and
a maximum of five within the simulation period.

It is worth noting that some of these clusters were not extremely
obvious, visually. They often separated areas of different agent
density within a larger cluster. However, it is still a useful metric
to differentiate a monoculture, in which all agents share highly similar
opinions, to one with some variation that changes over time.

### Bibliography

[**Individualization as Driving Force of Clustering Phenomena in Humans**](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1000959)

*Michael Mäs, Andreas Flache, Dirk Helbing*

This paper provides a mathematical model that reproduces pluralism
in social agents, where agents exhibit clustering behavior with
some degree of consensus within clusters.