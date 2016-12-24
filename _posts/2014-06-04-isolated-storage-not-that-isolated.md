---
layout: post
title: Isolated Storage ... not that isolated?
comments: true
tags: [C#, Development, Storage, Windows Phone]
categories: [Development]
---
A few weeks ago when I was at the <a title="http://fmendo.com/may-17th-publish-london" href="http://fmendo.com/may-17th-publish-london">//publish event in London</a> I had some trouble getting my code to work properly.<!--more--> You know, because the day you need to hack something together is the day the computer goes "I know what you want to do, but you're missing something, and I can't tell you what". Just another day at the office.

One of my biggest issues that day was that I was reading some data off of IsolatedStorage, and then doing some manipulation so the ViewModel could work it's magic, and that somehow managed to modify the data which I thought was safely stored in IsolatedStorage.

```csharp
object o = StorageUtility.ReadSetting(Utilities.PLAN_LOG);
var log = o as PlanLog;
var lastNWorkouts = log.Workouts.OrderByDescending(w > w.Date)
                                .Where(w => w.Exercises.Any(e => e.ExerciseName.Equals(config.TargetExercise)))
                                .Take(config.ConsecutiveFailCount);
var lastWorkout = lastNWorkouts.First().Exercises.First(e => e.ExerciseName.Equals(config.TargetExercise));
```

This is part of my <a title="WorkoutTracker on github" href="https://github.com/fmmendo/WorkoutTracker">Workout Tracking</a> library that I'm using in my <a title="fmendo: WP App - StrongliftsTracker" href="http://fmendo.com/stronglifts-tracker-app">Stronglifts app</a> and trying to extend into something (hopefully) quite useful. So, the first bit is quite simple. I have a little utility class I'm using to read and write to IS which I won't bother you with right now. I get the data and set a couple of LINQ queries to get the bits that I need.

```csharp
CurrentWorkout.ExerciseList.FirstOrDefault(e => e.Name.Equals(config.TargetExercise)).Sets = new List(lastWorkout.Sets)
```

My idea was that I had this data, fresh out of IS, and I wanted to copy it to my Model, which the ViewModel would then access to do it's thing. So rhat this bit of code was supposed to do was create a new List of Set, copying the elements from what I read from Isolated Storage. What was in fact happening is that I was creating a new list but the Set instances were referencing the original data. So when I then modified the data on my new list, the data in Isolated Storage would also get those changes (without me explicitly writing to it).

Now, as I side note, that code was in the published app and I swear I didn't notice anything misbehaving, but when debugging my app to show it off at the //publish event, any changes to the Sets list would affect the serialized data in IS. My suspicion is that since that Set class was a reference type, the CLR kept that "connection" alive, allowing changes to propagate to IsolatedStorage. So it's not as "isolated" as I believed.

In the end I had to modify that one line of code to ensure that I was created new instances of every item:

```csharp
for (int i = 0; i < lastWorkout.Sets.Count; i++)
{
    CurrentWorkout.ExerciseList.FirstOrDefault(e => e.Name.Equals(config.TargetExercise)).Sets[i] = new Set()
    {
        NumberOfReps = lastWorkout.Sets[i].NumberOfReps,
        SetType = lastWorkout.Sets[i].SetType,
        Unit = lastWorkout.Sets[i].Unit,
        Weight = lastWorkout.Sets[i].Weight
    };
}
```

... Yeah, I don't like that either. If you have a better idea I'm all ears.

That's all for now.
