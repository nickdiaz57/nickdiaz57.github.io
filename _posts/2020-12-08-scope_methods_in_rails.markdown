---
layout: post
title:      "Scope Methods in Rails"
date:       2020-12-09 00:07:08 +0000
permalink:  scope_methods_in_rails
---


Rails is incredibly powerful, almost uncomfortably so.  While working on this project I felt that every few minutes or so I was reminded all over again of just how much this framework does under the hood.  Although it's great that Rails can take care of so much behind the scenes, many times it made the coding process feel like trying to build a Lego set in another room by giving commands to a messenger.  I sort of felt like I had a grasp of what was happening, but so much could be done for me that I often wasn't able to pick up the pieces and puzzle out in my own way what exactly they were doing for myself.  Only with a strong fundamental understanding of the underpinnings of Rails (and countless trips through lessons and documentation) could I really get a feel for what the framework is actually building.

Writing a class scope method really brought up the need for a thorough knowledge of the basics underneath Rails.  In my application, I created a workout tracker, where users held a many-to-many relationship with the workouts through their reservations.  I wanted a way to display to a user all the reservations for workouts they were signed up for that hadn't happened yet, and a way to show them all the reservations they had already completed.  That would be possible with native Ruby methods, and it would probably look something like this:

```
@user.workouts.select {|w| w.date.to_date.future?}

@user.workouts.select {|w| w.date.to_date.past?}
```

While this might work for what I was trying to achieve, there were a few problems with this approach.  For one, the reservation model is largely cut out of this logic, and retrieving the reservation itself may become cumbersome depending on what specific data I'm trying to display to the user.  All the queries to the database would eventually start to slow the application down as it grew.  Iterating over this array would also make the code very messy very quickly.  Lastly, it simply just doesn't look very good.

Scope methods were a great way to achieve this logic in a cleaner, more streamlined manner.  We would start with a class scope method in the join table between the User and Workout tables:  the Reservation model.

```
scope :past_reservations, ->(user_id) {where(user_id: user_id)}

scope :future_reservations, ->(user_id) {where(user_id: user_id)}
```

This is a decent start; this will pull all reservations out of the Reservations table that have a ```user_id``` value that matches with whatever ```user_id``` we plug in to the method.  We're not there yet, though.  We want to retrieve the reservations based on the date the workout took place.  The great thing about these scope methods is that they can be chained together to further customize the database query that is sent.

```
scope :past_reservations, ->(user_id) {where(user_id: user_id).where("workouts.date < ?", DateTime.now)}

scope :future_reservations, ->(user_id) {where(user_id: user_id).where("workouts.date > ?", DateTime.now)}
```

This is close to what we're looking for, but there's a clear problem here.  The ```date``` attribute we're trying to filter by lives in the Workout table, not the Reservation table.  How can we query one table for rows filtered by an attribute on a different table?  By joining the tables together.

```
    scope :past_reservations, ->(user_id) {joins(:workout).where(user_id: user_id).where("workouts.date < ?", DateTime.now)}
		
    scope :future_reservations, ->(user_id) {joins(:workout).where(user_id: user_id).where("workouts.date > ?", DateTime.now)}
```

Here is where the knowledge of the foundations Rails is built on becomes so helpful.  By having knowledge of SQL and how its queries and database tables work, we know that joining two tables together along shared attributes will allow us to further refine our query and filter by attributes on both tables simultaneously.  So, adding ```joins(:workout)``` to our scope method will join the Workout table onto the Reservation table by its ```workout_id``` value.  Then, we can filter our results by the attributes of both tables; namely, the Reservation's ```user_id``` (which we plugged in to the scope method at the start), and the Workout's ```date``` attribute, which we compare to a current DateTime object to decide if the workout is in the future or in the past.

Now, with these scope methods working, we can write ```Reservation.future_reservations(1)``` to see all the upcoming reservations that User 1 has signed up for.  Or, we can write ```Reservation.past_reservations(2)``` to show User 2 data on all the workouts they've already completed.  This ends up being much cleaner and faster than the native Ruby methods we had before.

Understanding the foundations of Rails makes  a huge difference in helping to understand the true strength of the framework.  Sometimes taking a trip back to the beginning is a great way to help clear up the mysteries of the powerful tools we work with today.
