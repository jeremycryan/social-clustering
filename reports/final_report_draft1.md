# Social Clustering and Individualism
**Jeremy Ryan**

### Abstract
Within a population of humans, there tends to be a balance
of diversity and clustering of opinion — that is, for any 
continuous stance (say, degree of support for speed limit 
enforcement) there tend to be social clusters with some
amount of consensus within and some amount of variance
between each cluster. This phenomenon is called pluralism.

Based on prior work in this area, have implemented a Python
model to reproduce pluralism in an agent population.

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

### Next Steps

Now that I have implemented a basic version of the Mäs model, I hope to
accomplish the following things:

**Create more robust visualization** and validation of the initial model,
including reproducing a 3D heat chart from the paper that shows 
clustering behavior over various values of ```A``` and ```s```.

**Expand the model** to either include multiple axes of opinion (e.g.
agents have an ```opinion_a``` and an ```opinion_b```), or include
spacial information (agents are affected more strongly by agents with
a closer x/y position, or only agents that share an edge on a graph).


<!--### Experiments and Extensions-->
<!--For this project, I hope to-->
<!--- **Implement** the agent-based model detailed in the Mäs paper-->
<!--- **Reproduce** the results observed in the paper, including a-->
<!--plot of agent opinion over time that shows distinct clusters-->
<!--- **Expand** the model to include spacial relationships between-->
<!--agents, either through a graph representation or a model where-->
<!--agents move freely within a 2D space-->

<!--With my extension(s), I hope to gain more insight into opinion-->
<!--clustering with spatially-separated agents, where each agent is-->
<!--influenced more strongly by agents that are nearby than those that-->
<!--are far away. Tuning the strength of this effect may give-->
<!--results that parallel social effects as the Internet closes the -->
<!--"distance" separating existing groups and cultures.-->

<!--### Validation-->

<!--![image](https://github.com/jeremycryan/social-clustering/blob/master/images/clustering.png?raw=true)-->

<!--Shown above is a plot of agent opinion over time for a population-->
<!--of 100 agents, taken from the Mäs et al paper. For my directly-->
<!--replicated model, I hope to generate results that look similar.-->

<!--For the extension with spacial information, the 2D plot wouldn't-->
<!--be as useful. I suspect that I will end up creating something-->
<!--along the lines of an animated heat map, showing agent opinion with-->
<!--shades of color and demonstrating it changing over time.-->

<!--### Next Steps-->

<!--I plan to recreate the original model in the next week and a half or-->
<!--so, along with some tools to visualize it and tune some parameters.-->
<!--At that point, I hope to have more rigorously defined how I'll add-->
<!--spacial information to the model and can begin that implementation.-->

### Bibliography

[**Individualization as Driving Force of Clustering Phenomena in Humans**](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1000959)

*Michael Mäs, Andreas Flache, Dirk Helbing*

This paper provides a mathematical model that reproduces pluralism
in social agents, where agents exhibit clustering behavior with
some degree of consensus within clusters.