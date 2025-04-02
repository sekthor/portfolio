+++
title = "Please accept cookies; they're delicious"
date = 2025-04-01T18:04:16+02:00
tags = ["golang", "sqlite", "silly app"]
technologies = ["golang", "sqlite", "gorm", "javascript", "gin"]
+++

> Disclaimer: In case it should not be obvious enough, this is mostly a joke idea and meant to be seen as satire.

Since the GDPR of the European Union went into action in 2018, the average web surfing experience has changed significantly.
There are hardly any websites anymore, that do not require you to fill out their cookie consent form.

Instead of stopping websites from tracking users unless really necessary, the GDPR lead companies to enable cookie consent forms and continue business as usual.

Here's a thought:
I keep accepting cookies offered to me by a lot of different companies.
I have yet to receive any cookies in the mail.
Not a single one has thanked me for sharing my data with them by sending me pastries.

Let's think about that.

- They meticulously collect information about me
- They are only allowed to do this if I consent to accept cookies from them.
- I would not be surprised if one of the attributes tracked about me is my location. And thus, they most likely have my address.
- I am offered cookies which I accept.

**Where are my cookies?**

## Proposal

I propose that companies tracking user to a point they find out intimate details such as their address, should be required to provide the user with cookies in exchange for valuable data.

## Proof of concept

Just in time for April Fools, I have started implementing a proof of concept app.
The idea is, that upon accepting the cookie policy, the app will use a number of filthy tricks to attempt to obtain the users address.

So far I "supports"

- given name & family name form that includes hidden fields for addresses.
    - this is required to fill out to accept cookies
    - it expects users to be lazy and have that form auto filled.
    - should the browser have the users' address in the autofill properties, the hidden fields will be filled out and submit the users address without their knowledge.
- it will ask for the users permission to access the [geolocation](https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/getCurrentPosition) api.
    - It will then send the current location to the backend where it is evaluated for being a credible home address

## TODO

- implement more dirty tricks to obtain a home address.
- periodic checks of location (`watchPosition()`)
- implement a "order processing" process
    - messaging queue
    - order cookies
    - handle shipping

