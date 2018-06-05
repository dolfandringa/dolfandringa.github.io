---
layout: post
title: "Sighting Extractor: Using facebook for citizen science"
category: project
date: "2018-07-05"
tags: 
    - citizen-science
    - facebook
    - database
    - monitoring
    - data-entry
    - conservation
---

There are many citizen science projects around the globe where scientists or policy makers try to involve the general public in collecting data. For conservation, citizen science is a powerful tool, allowing policy makers and conservaitonists to involve the public in helping to make policy, employ a large voluntary workforce and create awareness among the public. Traditionally this has been done by creating dedicated mobile apps, which citizen scientists use to report data. A plethora of apps exist for many different purposes. 

The disadvantage of those apps is that they take up space and uses need to install them on their mobile device before being able to submit data. This requires a level of commitment that limits the size of the target audience. Furthermore, the provided forms are often inflexible. Especially tourists that are likely to only submit data once or twice, may not have the dedication to install and use these apps. A tool used by most of them though in their day-to-day life is Facebook. So why not use Facebook pages to let user report their citizen science information? The main disadvantage to using a free-form tool like Facebook is the lack of control over the way data is submitted. Using machine learning techniques like natural language and image recognition, it is possible to transform free-form reports from Facebook into the more strict data format needs of scientists.

SightingExtractor is a Facebook app that helps organizations and researchers involve users in their citizen science initiatives using Facebook instead of dedicated apps. Using the Facebook Graph API, posts to a page can be extracted, and the corresponding text and pictures used to extract citizen science data. Initial focus will be on extracting sightings of animals of marine biological interest to conservation organizations in the Philippines, but with additional work, the tools should be applicable in a much bigger field in the future.
