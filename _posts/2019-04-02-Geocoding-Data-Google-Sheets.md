---
layout: post
title:  "Geocoding Data in Google Sheets"
date: 2019-04-02
excerpt: "This post shows how easy it can be to geocode data using a custom script with google apps script in Google Sheets."
image: "/images/geocodegooglesheets1.jpg"
permalink: /blog/2019/04/02/Geocoding-Data-Google-Sheets
---

{% include advertisements.html %}


Many business people use spreadsheets. And many business people face the challenge of sometimes having to enrich their data with addresses and geo coordinates. However, many businesses don't have the skills, the time or the resources to utilise more technical bespoke solutions for geocoding and address enrichment / validation.


In this blog post I wanted to share a custom script I found online that helps non-technical users easily implement a geocoding solution in Google Sheets. As can be seen from the video below, after the script has been loaded, the user can easily reference a list of places, for example, in a spreadsheet and then run the scripts to populate date relating to XY coordinates (latitude, longitude) and / or addresses. 



The video below illustrates how it works once the script has been implemented:
<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed//Zp4zhCt7eew' frameborder='0' allowfullscreen></iframe></div>
<p> </p>



And below is the original script which was sourced from the following github repository [link:](https://github.com/nuket/google-sheets-geocoding-macro/blob/master/Code.gs)

<script src="http://gist-it.appspot.com/https://github.com/nuket/google-sheets-geocoding-macro/blob/master/Code.gs"></script>


More details on the Google Apps Script utilised can be found [here.](https://developers.google.com/apps-script/reference/maps/)
