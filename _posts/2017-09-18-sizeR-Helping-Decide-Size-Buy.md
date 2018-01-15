---
layout: post
title:  "sizeR - Helping you decide which size to buy"
date: 2017-09-18
excerpt: "In this blog post I wanted to build a simple solution to help online shoppers decide on the best size of clothes they should buy while online."
image: "/images/sizer1.jpg"
permalink: /blog/2017/09/18/sizeR-Helping-Decide-Size-Buy
---



In this blog post I wanted to build a simple solution to help online shoppers decide on the best size of clothes they should buy while online. 

Below is an interactive app built trying to solve this problem using [Shiny](http://www.shinyapps.io/) with R.

It is a much simpler version inspired by a really cool solution ZARA US online store uses to help shoopers decide which size are best for them based on their measurements and other shoppers' purchases. For more details on the ZARA US widget, go to the [ZARA](https://www.zara.com/) website.

In this example, since I didn't have actual shoppers data I created a mock dataset with modelled heights and weights and the associated sizes of each "shopper". It is far from polished and it will not cater well for outliers really high/low heights/weights that may exist in the real world. So if you want to break it, try and put some really crazy heights and weights and nothing will display.

A lot more can be done with this type of work to help shoppers decide on what purchase to make. I can also think of so many interesting benefits to companies' costs and brand if they adopt this type of solution. From savings related to less returned items to improved overall online customer experience.

Let me know your thoughts!

Note: In case you're reading this blog post from your mobile, this vis has also been published via the [shinyapp.io](https://feliperego.shinyapps.io/sizer_-_helping_you_decide_which_size_to_buy/) website which can make the experience in smaller screens a bit better.

<iframe src="https://feliperego.shinyapps.io/sizer_-_helping_you_decide_which_size_to_buy/" style="border: none; width: 900px; height: 750px"></iframe>
