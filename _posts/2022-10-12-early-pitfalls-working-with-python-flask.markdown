---
layout: post
title:  "Early Pitfalls Working With Python/Flask"
date:   2022-10-12
categories: bio
---
![image tooltip here](/assets/martin-sanchez-kaHZ4lS3eqs-unsplash-768x1024.jpg)

Early on in our Python/Flask project, I ran into a few problems, as tends to happen when you’re not fluent with a technology (Flask is relatively new to me). As I go over them one by one, I hope that you can learn something from me.

A couple of them had to do with how Flask works. First off, you can’t use:

```
flask run
```

from the command line to run the application unless the Flask application code is in a file called app.py. Ours, until we changed it, was in a file called main.py. No problem, we could also run it using:

```
python main.py
```

The next problem I had was a little hard to notice. Flask applications detect changes and restart the application automatically when you save those changes. The problem is that the restart doesn’t happen when the changes are in template files. I researched this and found that the documentation for Flask (search for TEMPLATES_AUTO_RELOAD here) claims that the detection of changes in template files is enabled by default in debug mode, but I found this not to be true in practice. I also tried to change the TEMPLATES_AUTO_RELOAD config variable, but it simply didn’t work, even though my changes to other config variables did. Therefore, I just have to remember to manually reload after template file changes.

Finally, there was a misunderstanding about our host, PythonAnywhere. I plugged in all the data about our host’s MySQL database into our app locally, but it simply would not connect. On researching it, I first confirmed that I had the information right, and I did. Ultimately, I found out that free accounts on PythonAnywhere cannot access databases on their server from anywhere but their server. This means I can do two things: access it through the MySQL console on their website, and theoretically, access it from our website once it is deployed to their servers. In the meantime, we will use a different database to operate locally and make sure we compose SQL files to create the tables and initialize data along the way so we can use those to set up our MySQL database on the host.

Those were my problems with Python/Flask and PythonAnywhere and how I dealt with them. Now you can avoid some of the bumps that I encountered. They are relatively small issues but can make a big difference if you don’t know about them.