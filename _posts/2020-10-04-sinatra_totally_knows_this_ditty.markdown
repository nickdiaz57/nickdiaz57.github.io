---
layout: post
title:      "Sinatra Totally Knows This Ditty"
date:       2020-10-05 03:00:46 +0000
permalink:  sinatra_totally_knows_this_ditty
---


Before I started seriously trying to learn programming, my first impression of it was an amusing interactive demonstration from a teacher pretending to make a sandwich while taking the class' directions as literally as possible, to try to imitate how computers do things.  The gist of the demonstration:  "Computers are pretty stupid, and they only will do exactly what you tell them to do."  I wasn't sure how true that was when I heard it, but working with Sinatra has really shown me that computers are a lot smarter than that demonstration made them sound.

Case in point: the session.

```
  configure do
    enable :sessions
  end
```

Three little lines stuck on the top of my application controller.  I added these to start taking care of the user authentication capabilities of my app, and for a while I thought these lines were all I needed.  I would drop into a binding, check the session, and a whole bunch of scary looking strings and numbers popped up, telling me the sessions hash did indeed exist.  Great!  Surely it's working as intended, right?

Nope.  Just when I thought I was doing fine, the bugs started creeping in.  Login checks around the app started failing, new instances of my models weren't being assigned to their proper users, and I couldn't persist my user's information like I thought I should be able to.  What's going wrong?  The session hash is right there, it should be working!  Enter: the session secret.

```
set :session_secret, "secret"
```

This little line was all it took to fix the issue.  I discovered that without setting the session secret, the program considered it to be such a security risk that it generated an entirely new session hash after every action I took in the program.  Generating a new session hash meant that every time I tried to save a user id to the hash and persist a user, it would be erased as soon as I took another action.  Then, a trip to the Sinatra documentation and one refresher on environment variables later, I settled on this:

```
ENV["SESSION_SECRET"] = 'SESSION_SECRET'

  configure do

    enable :sessions

    set :session_secret, ENV.fetch('SESSION_SECRET') { SecureRandom.hex(64) }

  end

```

And just like that, I had a program that persisted a user's information through a single session hash generated with a secure secret every time a user logged in.  It turns out, Sinatra was a lot smarter than I gave it credit for, so accounting for that made all the difference.  If Sinatra is already working this kind of magic, I can't wait to see what tricks Rails will have up its sleeves.
