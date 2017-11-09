---
layout: post
title:      "Building a Knicks Themed CLI Ruby Gem "
date:       2017-11-09 20:39:08 +0000
permalink:  building_a_knicks_themed_cli_ruby_gem
---

![](https://pbs.twimg.com/media/CmSGapkWAAAelZz.jpg)         ![](https://avatars1.githubusercontent.com/u/208761?s=400&v=4)

For my first portfolio project, I set out to code a ruby gem that would scrape information about the New York Knicks. The intention of this projects was to build a an object-oriented program that utilizes Nokogiri in order to populate those objects. In order to interact with the data, a simple command line menu was also implemented. You can access my repository [here](https://github.com/dkStephanos/KnicksHistoryCLI).

This is my first time building a ruby gem, so the process began with a lot of reading and video tutorials, but eventually I was ready to start building and the first step involved an already existing ruby gem, bundler. Bundler maintains your coding environment and will even create the basis structure of a Ruby gem for you rather painlessly. All you have to do is install bundler and run the following line of code, replace 'gemName' with the name of your gem:

```
bundle gem "gemName"
```

Now that we had the basic structure, it was time to create some classes and implement their functionality. Since I was looking at historical stats for the Knicks, I chose to arrange my data in Season objects which are a metaphor for an actual basketball season. We also need a CLI class for our menu and selecting options. The last main addition is a scraper class, which is responsible for obtaining the actual data used to populate our Season objects. I used [basketball-reference.com](https://www.basketball-reference.com/teams/NYK/) as my source, because they had this nice formatted table. Each individual row represents one season, and the html for a row looks like this:

```
<tr data-row="0"><th scope="row" class="left " data-stat="season"><a href="/teams/NYK/2018.html">2017-18</a></th><td class="left " data-stat="lg_id"><a href="/leagues/NBA_2018.html">NBA</a></td><td class="left " data-stat="team_name"><a href="/teams/NYK/2018.html">New York Knicks</a></td><td class="right " data-stat="wins">6</td><td class="right " data-stat="losses">5</td><td class="right " data-stat="win_loss_pct">.545</td><td class="right " data-stat="rank_team">4</td><td class="right " data-stat="srs">-1.08</td><td class="right " data-stat="pace">96.7</td><td class="right " data-stat="pace_rel">-2.3</td><td class="right " data-stat="off_rtg">108.9</td><td class="right " data-stat="off_rtg_rel">2.4</td><td class="right " data-stat="def_rtg" csk="109.9">109.9</td><td class="right " data-stat="def_rtg_rel">3.4</td><td class="left " data-stat="rank_team_playoffs" csk="0:0:2018"></td><td class="left " data-stat="coaches" csk="Hornacek,Jeff.2018"><a href="/coaches/hornaje01c.html">J. Hornacek</a> (6-5)</td><td class="right " data-stat="top_ws"><a href="/players/p/porzikr01.html">K. Porzingis</a>&nbsp;(1.4)</td></tr>
```

The Season class is initialized with a hash of values. The values correspond to attributes stored as instance variables within each individual season. The initialize function, as a result, simply takes each hash value and stores it in its corresponding instance variable before finally passing the newly created season into the class variable @@all which is of type hash. I chose to store the seasons in a hash as opposed to an array for searching purposes. Since each NBA season is necessarily tied to a specific year, the key/value format seemed most useful. Which allows the following method to work:

```
def self.find_season(year)
    @@all.detect do |season|
      season[0] == year.to_s
    end
  end
```

The Scraper class has only one responsibility really, and that is creating the hash of values from the website used to instantiate the Season. It only has three methods, the first of which opens the webpage and the second selects all the rows in our table. From there, our third method, make_seasons, uses css selectors in order to parse the data from the table and save them in a hash with the correct key values. This hash is then passed as an argument to create a new Season object. The method loops through every row doing this, and when it is done, we now have a collection of Seasons as hashes with key/value pairs for their attributes, stored within a hash containing the entire history of the New York Knicks. Here's a peek at what that method looks like (note: defensive and offensive ratings weren't recorded for the first few seasons, hence we have to check to make sure that data is present in the table, saving the string 'N/A' for not applicable if not):

```
def make_seasons
    data_hash = {}
    scrape_seasons_index.each do |season|
      data_hash[:year] = season.css("th a").text.match(/\d{4}/).to_s
      data_hash[:wins] = season.css("td[data-stat = wins]").text
      data_hash[:losses] = season.css("td[data-stat = losses]").text
      data_hash[:win_percentage] = season.css("td[data-stat = win_loss_pct]").text
      if data_hash[:off_rating] = season.css("td[data-stat = off_rtg]").text == ""
        data_hash[:off_rating] = "N/A"
        data_hash[:def_rating] = "N/A"
      else
        data_hash[:off_rating] = season.css("td[data-stat = off_rtg]").text
        data_hash[:def_rating] = season.css("td[data-stat = def_rtg]").text
      end
      data_hash[:best_player_ws] = season.css("td[data-stat = top_ws]").text
      KnicksHistory::Season.new(data_hash)
    end
  end
```

So at this point, we are creating and storing all the data we need, and have formatted it in such a way that it may be retrieved by specifying a year. This means that our CLI only has to read input and pass that to our Season class search method to retrieve all the information we want so that it can be printed to the screen, giving us a nice formatted little snapshot of an individual season, like this:

```
Year  Wins  Losses  Win%
2014   17     65    .207
```

After the basic season stats, the CLI gives another prompt for more specific info if desired, and outputs that to the screen like so:

```
Other Stats (enter the number of the statistic you want to see or type back)
1. Offensive Rating
2. Defensive Rating
3. Best Player w/ Win Shares
3

Best Player(WS)
C. Anthony (9.5)
```

Now that my gem is operating the way I intended it to, its time to publish and submit to RubyGems! (Happy Melo)

![](https://typeset-beta.imgix.net/elite-daily/2016/02/15234253/MeloTat.jpg)

