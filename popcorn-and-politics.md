---
layout: page
title: Popcorn and Politics
year: 2009
project: true
--- 

![Popcorn and Politics in action][1]

Popcorn and Politics is a data visualization of the relationship between movie-goers and American politics. It is based on the idea that money spent on a cinema ticket finds its way to the film's producers as their primary source of revenue, which they then can use to donate to politicians. Popcorn and politics reveals the surprising places the viewer's ticket money can find itself. It was completed as a collaboration with designer [Aaron Druck][3] for [Alexis Lloyd][4]'s Data Visualization class at Parsons the New School for Design. The site and documentation are now defunct.

The viewer enters the name of a movie in the text field and hits return. This adds the movie's name to the top row of the visualization and draws lines of varying thickness down to the names of 2008 presidential candidates, representing how much money that movie's producers donated to that candidate in the style of a [sankey diagram][5]. The database was built as a mashup between FEC data, Wikipedia, IMDB, Film-releases.com, and the NYTimes API. The client is written in Flash and the server is written in Ruby using Sinatra.

[1]: http://nas.sr/popcorn-and-politics/shot.png
[3]: http://www.whatthedruck.com/
[4]: http://www.alexislloyd.com/
[5]: http://en.wikipedia.org/wiki/Sankey_diagram