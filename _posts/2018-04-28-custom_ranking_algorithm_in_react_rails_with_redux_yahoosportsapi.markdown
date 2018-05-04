---
layout: post
title:      "Custom Ranking Algorithm in React/Rails with Redux/YahooSportsAPI"
date:       2018-04-28 15:59:46 -0400
permalink:  custom_ranking_algorithm_in_react_rails_with_redux_yahoosportsapi
---

![](https://pbs.twimg.com/profile_images/719644031144210432/w3IT_JOM_400x400.jpg)
# What is Punting?

In the world of fantasy basketball, there is one particularly dominant strategy, punting. Punting, you might ask? Isn't that something you do in football, not basketball? Well, aren't you the clever one. It is true, punting the ball mid basketball game is frowned upon, and will usually get you ejected. (See Tracy McGrady)

![](https://i.makeagif.com/media/10-12-2015/LdXjuR.gif)

So what does punting mean in this context? Well the majority of fantasy basketball leagues are something called 'category leagues'. This means that a fantasy 'game' involves competing to win one of typically 8-10 statistical categories (like points, rebounds, assists, etc.). It is a general rule of life that it is easier to be great at a couple things, then good at everything, and this holds true for fantasy. Being competitive in 8 or 9 different statistical categories is almost impossible, so many users end up picking a category or two to concede in favor of being better at the remaining categories. The categories the user chooses to concede are the 'punted' categories. 

Okay, so now that that's out of the way, what is the goal of this project? Our goal is to provide a way for users to compare players without the specified category they chose to punt. This is actually pretty involved, because we can't rely on the algorithms that the fantasy sports provider (Yahoo Sports in this case) uses to rank their players. So, the first big step is to write our own player ranking algorithm we can tweak and filter as we see fit.

# The Formula
So, first things first. What is this formula? NBA Abacus has a pretty good [blog post](https://nbaabacus.wordpress.com/2013/10/20/the-yahoo-fantasy-nba-ranking-formula/) demonstrating the pattern that Yahoo Sports uses to rank its own players, and this is going to be our inspiration. 

Ultimately, we need a way to compare a player's fantasy impact across categories, and the best statistic for determining how above or below the mean a given statistic is, is the z-Score. A z-Score is literally the amount of standard deviations from the mean of a given stat. So, based on that, we are going to need to find out the mean and standard deviation for each statistical category, so we can calculate each player's z-Score in that category. Here is the formula for z-Scores:

![](https://i.pinimg.com/originals/59/66/25/596625c0eec9f027d019bd86cd6e05b1.jpg)

One issue with this approach is that basketball statistics aren't just counting stats like points, rebounds, and assists, we also have to account for percentages like field goal percentage (FG%) and free throw percentage (FT%). Finding the z-Scores for these percentages is not sufficient, because it doesn't account for quantity. In order to properly rank these percentages, we will need to account for the number of attempts, and this is called the field goal or free throw 'impact'. This [reddit post](https://www.reddit.com/r/fantasybball/comments/71bdq0/how_to_calculate_weighted_zscore_for_fg/) actually has a great break down on obtaining these values.

Finally, once we have calculated the shooting impact and z-Scores for each category, we simply take the average of those scores. This average essentially reflects how above or below average a given player is across all of the relevant statistical categories, and is what we compare to get our final player rankings.
# Fetching the Data
Okay, well now that we have an idea of how to get our player rankings, we can turn our attentions to obtaining the player statistics. We will be using the Yahoo Sports API to obtain this data, and there are a few advantages to this route. 

One pro is that we can use Yahoo's API for our user authentication, returning a JSON Web Token (JWT) with which we can then fetch player data. The big advantage to fetching player data from the fantasy league of the user, is that we get back only the stats for the categories of that league. So for example, if the user was in a league that had 8 categories, we would get back stats for just those 8 categories, which makes our ranking algorithm all the more user specific, which is definitely what we want.

So once the user logs in, we prompt them to select the team/league they want to get data from. Then using the user specified league key, we dispatch a Redux action to our Rails backend for the data, including the user's JWT in the headers of the request:

![](https://i.imgur.com/gMdfsHV.png)

Rails then parses out the JWT and the league key and submits a GET request to the Yahoo Api for the player data, converting it from XML to JSON (unfortunately), and then rendering the player data for our React front end. Yahoo's API will only return a max of 25 players at a time (we want around 300 for a good sample) so we will have to submit multiple requests:

![](https://i.imgur.com/rtxWGHT.png)
# The React Component
Great! So at this point, we know how to get the stats we need, and we know how to obtain our custom player rankings, so we need to wrap all this up in a React component so we can integrate what we have learned into our application. Since there is some overhead involved in fetching all that player data and running our statistical analysis, I chose to place all of these actions in their own component which doubles as a loading screen. 

In order to ensure we perform the steps in our ranking algorithm correctly, while storing our results in the Redux state, we are going to utilize the React component lifecycle methods. These methods (which are a part of every React component) allow us to essentially 'pause' the execution of our application at specific moments. The first of these we will use is 'componentDidMount' which is exactly what it sounds like. Whenever the given React component is injected into the DOM, this lifecycle event triggers, and we can perform our relevant actions, in this case, fetching our player data.

The second lifecycle event we need to utilize is 'componentDidReceiveProps'. Props (or properties) are the data specific to a component, and in our case, are linked to the Redux state. That means, when we fetch the player data and store it in state, it updates the props of our component, and triggers this lifecycle event. This is where we will do our statistics and the bulk of our player ranking. Now, z-Score is dependent on standard deviation, and standard deviation is in turn dependent on the mean which requires the field goal/free throw impacts... So basically, we have to be careful that these occur in a specific order, only performing one step of the algorithm at a time, updating Redux state as we go, so we can hit our lifecycle event and perform the next operation, redirecting to the home page when finished. While this looks more involved, it actually happens a lot faster than simply fetching the player data to begin with. 

Here is a look at the playerFetchTransition component lifecycle events:

![](https://i.imgur.com/O67FAlc.png)

# The Redux Actions
Now that we can see how each step of the algorithm is performed in a React component using Redux for state management, lets go down a level of abstraction, and take a look at each step of that process. 

We begin by calculating the field goal/free throw impacts. This is done by finding the average shooting percentages for all players, subtracting that mean from the percentage of the individual player, and multiplying the result by the amount of attempts the player took. What we end up with doesn't really reflect a shooting percentage, but is a better indicator of how a player's shooting impacts their team and compares with other players who may average more or fewer attempts. The first half of the algorithm determines the average shooting percentages:

![](https://i.imgur.com/NzI0RIU.png)

The second half then calculates the field goal/free throw impact, and pushes the result (as an object with a stat_id for output) into the player stats array, finally dispatching an action creator that will store our updated players array in state:

![](https://i.imgur.com/rA8FKS4.png)

Now, each player's stats array has all the values we need for ranking, so we can move on to getting the means. Both the means and standard deviations are statistics that relate to all players, not any player in particular, so instead of storing them with our player data, we are going to use a seperate set of Redux actions with their own stat reducer. The formula for the means is pretty straight forward. We initialize the mean of each category to the values of the first player, then loop through the rest, adding the stats, finally dividing each total by the amount of players (we ignore the composite stats of FGM/FGA and FTM/FTA as they are only relevant to the shooting impacts we already obtained):

![](https://i.imgur.com/fdo0Jy3.png)

The standard deviation formula is a bit trickier and requires some exponent work, but follows a very similar pattern:

![](https://i.imgur.com/f55TSxN.png)

Now that we have our shooting impacts, means and standard deviations, we can finally calculate the z-Scores! This is the implementation of the formula mentioned earlier. Since we need to store the z-Score for each stat for each player, we are going to add the values directly into the player stats array, storing our new player data in state:

![](https://i.imgur.com/jsN7H65.png)

Finally, we rank! As mentioned earlier, our player 'rank' is really just the average z-Score for a player. We then compare players based on this 'rank' to determine their actual numerical rank (number 3 overall, for example). Yet again, this number is extremely relevant to each player, so we will store it with our player data under a new key, 'rank':

![](https://i.imgur.com/S75mDaO.png)

Here is a look at what the player data object for LeBron James looks like once we are done with all of these steps. Notice that we have two stats at the end of the array with keys of '1005' and '1008' which correspond to our shooting impacts, that each stat has its own z-Score stored alongside it, and that their average is stored under our new key of 'rank':

![](https://i.imgur.com/Y3Ha4Li.png)
# The Player Rankings
Whew! That was a lot, I know, I had to write it, but we are almost done! We have done all the real number crunching required for our rankings, and we have been redirected to our home page (storing our redux state with [redux-persist](https://github.com/rt2zz/redux-persist)), where we will show the user all the work we did. A couple things need to happen first in order for us to do that, however. 

Remember that depending on the setting of the user's league, we could be calculating totally different categories for one user as opposed to another. This means we have to dynamically create labels for our table. Yahoo Sports stores all their player stats under numeric stat keys (and as a result, so do we). This allows us to know exactly what stat we are working with at a given time, but means there is an extra step in converting those keys to the labels our user expects. For example, it would be better to label our points column 'Points' as opposed to '12'. So when the homePage component mounts, we need to determine category labels, storing the result in state, like so:

![](https://i.imgur.com/7R864Su.png)

We also need to sort the players array based on the rank we calculated earlier, this action is also triggered from the componentDidMount lifecycle event of the homePage component and looks like this:

![](https://i.imgur.com/RzwGCiu.png)

When these actions are completed. We can use our category labels to create the table headers. The homePage component can then loop through the players array in its properties, creating PlayerRow components with the relevant stats for each player, finally rendering our table of players, in order, based on the ranks we calculated! Here is a look at that component:

![](https://i.imgur.com/y5oplPj.png)
# Conclusion
We did it!

![](https://i.imgur.com/LXGFWe4.png)

We now have our very own custom player ranking algorithm implemented in React, using Rails for calls to the Yahoo Sports API and Redux to store our calculations and player data. While this is not the ultimate goal of this project, it is a very necessary first step. In order to make this relevant to our user that wants to punt a specific category, we need to add a way to filter our algorithm, to only take into account the statistical categories which are not being punted. 

Feels like that would make a great blog post sequel... 
