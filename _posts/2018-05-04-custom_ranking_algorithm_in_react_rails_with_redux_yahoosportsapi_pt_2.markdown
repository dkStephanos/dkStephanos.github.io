---
layout: post
title:      "Custom Ranking Algorithm in React/Rails with Redux/YahooSportsAPI, Pt. 2"
date:       2018-05-04 20:16:43 +0000
permalink:  custom_ranking_algorithm_in_react_rails_with_redux_yahoosportsapi_pt_2
---


![](http://s3.amazonaws.com/dev-pablo.marketkarma.com/yahoo-nba-fantasy-basketball-art-work.jpg)

# Where we Left Off
So this is a sequel to [this](http://stephanossolutions.com/custom_ranking_algorithm_in_react_rails_with_redux_yahoosportsapi) blog post. This should still make some sense without reading it, but in case you were curios how we got to where we are, or what in the heck punting is, hopefully all those questions can be answered there! 

So, where did we leave off? We wanted to write our own ranking algorithm for fantasy basketball, so we could help out with punting strategies. Briefly, a punting strategy is one in which a fantasy owner drafts players with a similar weakness in order to focus on the other statistical categories, making them more competitive. Typical ranking algorithms for fantasy sports rank across all categories at the same time, but if you are eliminating a category from contention, these rankings can vary dramatically. 

In the last blog post, we covered fetching the player data, processing the stats, calculating z-Scores for each category for each player, and averaging them for our ultimate player rank. We then sort based on this z-Score average and rendered our list of players. In this blog post, we are going to build on this, by adding category, position and health status filters that we can apply to adjust our rankings.

# Adding Filter Checkboxes
So first things first, it's going to be hard to apply these filters without a UI. I am using [react-bootstrap](https://react-bootstrap.github.io/) for some simple components, so we will import our checkboxes from there. Since we have three different types of filters, and we want to interact with them from our 'homePage', we will need to do a little bit of set up for that to work. 

First off, we are going to need some sort of internal state to keep track of which checkboxes are checked at a given time. Secondly, we need a way to submit these filters, which will occur via a callback function. To accomplish this, we are going to have an internal function to keep track of each set of checkboxes, and we are going to bind all of these, along with our callback function, in the component constructor. Here is a look at the beginning of our component:

![](https://i.imgur.com/fLT6K5A.png)

Cool, now that that is out of the way, we need to create the checkboxes. For the status and position filters, we can hard code to an extent, because player positions and health statuses don't change. The stat categories are going to be different depending on the user's league settings, so we need to generate those abstractly. We have available to us all of the 'categoryLabels' we generated earlier, so we can trust that contains everything we need. It also has some stuff that isn't relevant to our ranking algorithm like the player name, team, etc.. For that, we will create a blacklist of categories to ignore, and generate checkboxes from what remains. Here's how we do that:

![](https://i.imgur.com/5Sd6fLm.png)

Now that we have the checkboxes we need, generated for just that categories we are concerned with, we wire everything up, and return our checkboxes. Remember, we need to pass each collection of checkboxes a callback function for status changes, so we can monitor what is/is not selected at a given moment. We also need to pass the callback function we received from 'homePage' to the 'Apply Filters' button. Slap a label on each set of checkboxes and we're done:

![](https://i.imgur.com/aYTxxM8.png)

# Adding Filtered Players to Redux State
Okay! Making progress, now we just need somewhere to put our players once we apply the filters in our fancy new checkboxes. Currently, we have an array of players called, you guessed it, 'players', stored in Redux state. This is where we pull our stats and data from, and where we store things like our z-Scores and player ranks. When we filter players, we want to be careful we don't lose any data we might want later. Calls to external API's are costly, so we aren't going to touch our 'players' array when filtering. Instead, we are going to add a new array to our Redux state called 'filteredPlayers'. I know, who could have seen that coming? The advantage to this approach, is we can apply multiple filters in a row, sorting and resorting, and still revert back to our original players array without fear of data loss. 
# Ranking Algorithm with Filters
At this point, we have our ranking algorithm, our player data, checkboxes that pass filters to a callback function and a collection of 'filteredPlayers' in state just waiting for some players. Almost there, now for the hard part. We need to write a new version of our ranking algorithm that will take in an object containing our arrays of filters, sorted by type. Of these types, there are basically two sub-categories: those that remove players from the algorithm, and those that remove stats from the algorithm. This means that we will want to filter out the players first, so when we rank and sort, we only include the players we want, and don't compare stats to those of players we will later filter out.

First things first, however, a little setup and maintenance. In the start of our algorithm, we initialize 'rank' and 'playersLength' for later, and most importantly, convert our stat categories. The actual player stats array uses numeric keys for the various categories, so we need to convert our string labels to their numeric counterparts, which is achieved like this:

![](https://i.imgur.com/cCE38Cr.png)

One thing we want to consider throughout, is the possibility we don't have certain types of filters, so we need checks before we start processing our filters, to make sure we aren't trying to access data that doesn't exist. Keeping that in mind, we can move on to the player and status filters. If we have them, we run through our array of players, pushing only those that do not match one of our selected filters to the filtered players array. Here is that section of the algorithm: 

![](https://i.imgur.com/GOvMSYm.png)

Now that we have the exact players we want, we can calculate their ranks like we did before. The only catch this time, is we are going to ignore any stat in our filters. Recall that our player rank is really just an average of the players z-Scores across the categories within the league. So by removing a given category from contention, what we really mean is don't include it in the calculation of the mean. When we are done calculating the new ranks, we dispatch a new action that will store our collection of newly filtered and ranked players in our 'filteredPlayers' array in Redux state. Here is the end of the algorithm:

![](https://i.imgur.com/hOgodQn.png)
# It Works!
That's it! We now have everything we need for our customized ranking algorithm, so let's see it in action. First off, let's take a look at what our top eight players look like without filters:

![](https://i.imgur.com/QgOG8xO.png)

As we can see, LeBron sits perched clearly on top for now. Westbrook is third overall, and we round out with Giannis, the Greek Freak! This is very similar to the actual Yahoo! rankings, which makes sense. We are using a smaller sample size so can expect some variation, but not too much. Lets apply some filters:

![](https://i.imgur.com/kLqWMK0.png)

First off, a couple stat filters. This means we are no longer including the amount of points or triple doubles a player accumulated, which translates to removing the z-Scores for these values from the average, or player 'rank'. Here are our top eight players after we apply these filters:

![](https://i.imgur.com/QgOG8xO.png)

As we can see, there is some notable player movement. LeBron remains unwaveringly the most valuable, but Westbrook has fallen completely out of the top eight. This makes sense, because Westbrook is by far the league leader in triple doubles, and among the league leaders in points. As a result, much of Westbrook's value in our algorithm is derived from these two categories. 

Conversely, a player like Towns jumps all the way up to third. This is because Towns, while still a good scorer, didn't notch a single triple double on the year. So while removing that category really hurts someone like Westbrook, it doesn't affect Towns at all! This allows him to gain grown compared to his peers, and is a reason why you might select him over Westbrook if you were punting triple doubles, a decision that would make little sense without such a condition.

That is not enough though, we have more filters to play with! Next let's add some position filters so we can see some new players on top:

![](https://i.imgur.com/i9eHQKo.png)

We are now filtering out small forwards and centers from our rankings. Remember, this doesn't just mean that we aren't showing those players, but that we are actually ranking the remaining players against JUST each other. Meaning if we were to filter out everyone but point guards, we would be left with a list of point guards ranked against only other point guards. This is helpful for certain reasons, like, say you need to draft a center, and don't care how a center compares to a small forward, because they should, by definition, be good at different things. Keeping that in mind, here are our top eight players after the position filters:

![](https://i.imgur.com/cbkAEz5.png)

Guard dominance! Removing anyone with small forward or center eligibility leaves us with mostly guards. When ranked against each other, there are a few things to note. One, Westbrook is back! Two, Harden is now the number one ranked player, because even without points or triple doubles, his excellent efficiency and assists make him dominant. Lastly, Kyrie appears in our top eight for the first time, which is notable, because he is the first player currently injured to do so. So let's add another filter and remove any player currently injured.

![](https://i.imgur.com/GyrpcW6.png)

And our top eight:

![](https://i.imgur.com/EP5ouAZ.png)

Goodbye, Kyrie! Not much else changes, and that is to be expected, but what is worth noting, especially when it comes to a large pool of players, is that although Kyrie is the only one we saw leave, everyone's rank is changed, because they are no longer being compared to injured players. This is one of the cool things about ranking after we filter players out. Yahoo, as far as I can tell, includes all players at all times. So if you are trying who to decide if you should pick a player ranked 245 or 247, it would help if those ranks weren't influenced by players who aren't playing or play a drastically different position. This allows us to do that, in addition to our punting analysis.
# Conclusion
Congratulations, we did it! We now have the exact functionality we set out to build in the last blog post. We are pulling in a collection of players from Yahoo Sport's API, using our own custom ranking algorithm to rank players with our without a number of statistical, positional, or status related filters. Obviously, there is a lot more we can do with this, like adding filters for player ownership within the league, ranking based on average stats as opposed to cumulative, adding player advice sections that indicate a players strengths and weakness compared to their peers... Honestly I could keep going and going, but maybe there will just have to be another blog post at some point. ;)
