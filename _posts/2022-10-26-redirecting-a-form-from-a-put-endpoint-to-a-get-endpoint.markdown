---
layout: post
title:  "Redirecting a Form to a PUT Endpoint and Back to a GET Endpoint with jQuery When Using Flask"
date:   2022-10-26
categories: tutorial
---
![image tooltip here](/assets/greg-leaman-K25hwSB2NuE-unsplash-2048x1365.jpg)

As someone who has been most comfortable working on the back-end in recent years, I was bound to run into a problem or two when writing front-end code. I ended up struggling with forms, but I found the solutions in a fun and functional little language called jQuery, which came to the rescue!

The overarching problem I had was that I wanted to submit a form to a PUT endpoint. This is not natively supported; forms can only submit to POST or GET endpoints. However, I was editing a resource with this form and I couldn’t see myself doing that with anything but a PUT endpoint (I worked on an API for three years, so I’m pretty attached to its rules), so I searched for a way around it. That’s when I stumbled upon jQuery. It can override the default functionality of the form and submit the data to any endpoint you want. I scoured the web, and here’s what I ended up with:

```javascript
1       $('#updateProfile').submit(function (event) {
2            event.preventDefault();
3            $.ajax({
4                 url: '/profiles/{{ data['profile'][0] | tojson }}',
5                 datatype: 'json',
6                 type: 'PUT',
7                 data: new FormData(this),
8                 processData: false,
9                 contentType: false,
10                success: function(data, status, xHR) {
11                     window.location = '/manage-profiles'
12                }
13           });
14      });
```

Now, I’m no expert in jQuery, so I didn’t get this all right off the bat. Everything went wrong from the code all being ignored, to not all the form data being serialized, to the success function being disregarded. But this is how to do it right, at least as of this writing (the most challenging thing about jQuery is that syntax seems to become obsolete relatively rapidly when compared to other languages, so out of date tutorials abound). Let’s walk through it in order to give you a better understanding of the individual parts.

Line 2 prevents the form from executing its default action (that is, submitting itself to the get method of the current URL, just adding all those query strings).

```javascript
2            event.preventDefault();
```

Line 7 shows how to include the form data. There’s also a serialize() method, but this does not serialize file inputs, so it didn’t work for me in this case and is probably better to avoid in general for that reason. Lines 8 and 9 go with the FormData method of serialization.

```javascript
7                 data: new FormData(this),
8                 processData: false,
9                 contentType: false,
```

Line 12 shows the success function. Importantly, the endpoint you sent the function to must send back JSON of some kind or it won’t work. I was using an empty string and couldn’t figure out why I was never getting to the success method. The second block of code below shows the return line from my PUT endpoint in Flask. As you can see, the JSON doesn’t have to be anything fancy, but it does have to be JSON.

```javascript
12                success: function(data, status, xHR) {
13                     window.location = '/manage-profiles'
14                }
```

```python
1  return { "Result": "Success" }, 200
```

You might be wondering why I’m using the success function for redirecting when I’m using Flask. Flask does redirects, right? Well, unfortunately, when you use jQuery in this way, Flask can apparently only redirect you to the same method type. So my jQuery sent you to a PUT endpoint; therefore, any redirect I try to do within Flask will take you to the PUT method of that URL, which, in my case, does not and should not exist. I researched this extensively and was unable to find a fix on the Flask side. Fortunately, redirecting via jQuery worked fine.

I hope you have found this helpful. I did consider that perhaps the ‘right’ thing to do here would have been to just make the endpoint a POST one; that’s up for debate. However, if, like me, you were looking for the way around this in 2022, now you’ve found it.