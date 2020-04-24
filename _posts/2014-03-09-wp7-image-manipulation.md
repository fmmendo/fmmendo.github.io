---
layout: post
title: WP7 Image Manipulation
comments: true
tags: [C#, Development, Gestures, Windows Phone, Xaml]
categories: [Development]
---

I decided to give my WP7 app a much needed overhaul and eventually release a WP8 version. I started by pulling all the Model and ViewModel code to a separate library so I can then have an app for 7 and 8. One of the features I planned to update for a while but never got to it was to have the image viewer actually work like an image viewer.<!--more--> (If I did my research correctly, I believe this will be a non-issue in WP8 as there's a control for what I'm about to write about).

Note: There are loads of controls out there that do this kindly provided by other developers, or from sources such as Telerik. But I do like to reinvent the wheel sometimes and get to know how things work.

My first attempt was quite obvious and was to use the GestureListener from the Windows Phone Toolkit (and to be fair it worked, just not exactly how I wanted). Basically we would use the DragDelta and PinchDelta events to calculate, pan and zoom, and apply the transformation. But I settled for the ManipulationStarted, ManipulationDelta and ManipulationCompleted events. Let's have a quick look at my Image control:
<pre class="brush: html;">      
        
          
           
           
          
        
</pre>
First, the RenderTransforms. This is what will be handling the transformation matrices for the current element, whether you're rotating, scaling, translating, etc. In this case I need the Translate and Scale Transforms. The RenderTransform doesn't like multiple items, so we pack these nicely in a TransformGroup, and I'll get back to them later.
<pre class="brush: html;">        
          
            
              
                
              
            
            
              
                
              
            
          
        
      
</pre>
Now for that Storyboard taking up half the code. While not entirely needed, it looks nice, and if you use the gallery app you'll notice that there's a bit of inertial animation going on, so this will (eventually) perform that deceleration.

Now all we need to do is get the code to actually work. Onto the code behind!

The first bit is easy, you pick up the DeltaManipulation for either a scale or a translation, apply the transform and it's done. After playing around in the app you lose your image by dragging it off-screen and it's back to the drawing board... so let's have a look at the code to make it behave nicely:

```csharp
    private void photo_ManipulationDelta(object sender, ManipulationDeltaEventArgs e)
    {
      if (e.DeltaManipulation.Scale.X != 0.0 && e.DeltaManipulation.Scale.Y != 0.0)
      {
        double tmp = ScaleTransform.ScaleX * e.DeltaManipulation.Scale.X;
        if (tmp < 1.0) //min
          tmp = 1.0;
        else if (tmp > 4.0) // and max
          tmp = 4.0;
        ScaleTransform.ScaleX = tmp;
        ScaleTransform.ScaleY = tmp;
      }
```

Ok, so for scaling we're reasonably safe, I'll just add a minimum and maximum zoom to avoid some disasters. Moving on:

```csharp
      else
      {
        Image photo = sender as Image;
        var transformgroup = photo.RenderTransform as TransformGroup;
        var transform = transformgroup.Children.First(c => c is TranslateTransform) as TranslateTransform;
        if (transform != null)
        {
          // Compute the new X component of the transform
          double x = transform.X + e.DeltaManipulation.Translation.X;
          double y = transform.Y + e.DeltaManipulation.Translation.Y;
```

So first we need to get the current transformation, and add the new changes, nothing fancy. The interesting bit is what comes next. We want to keep the image from going out of bounds, and we want the user to be able to pan around a zoomed in image. One way to do it is to have its movement limited by its edges, like the building gallery does. So when one drags and image to the right and its left edge reaches the left edge of the screen, there's no point in moving it any more. So we end up with this:

```csharp
          // going left
          if (e.DeltaManipulation.Translation.X < 0)
          {
            if (Application.Current.Host.Content.ActualWidth - photo.ActualWidth * ScaleTransform.ScaleX > 0) return;
            if (x * ScaleTransform.ScaleX < Application.Current.Host.Content.ActualWidth - photo.ActualWidth * ScaleTransform.ScaleX)
              x = (Application.Current.Host.Content.ActualWidth / ScaleTransform.ScaleX) - photo.ActualWidth;
          }
          // going up         
          if (e.DeltaManipulation.Translation.Y < 0)
          {
            if ((Application.Current.Host.Content.ActualHeight - 206) - photo.ActualHeight * ScaleTransform.ScaleX &gt; 0) return;
            if (y * ScaleTransform.ScaleX < (Application.Current.Host.Content.ActualHeight - 206) - photo.ActualHeight * ScaleTransform.ScaleX)
              y = ((Application.Current.Host.Content.ActualHeight - 206) / ScaleTransform.ScaleX) - photo.ActualHeight;
          }
          // going right
          if (e.DeltaManipulation.Translation.X > 0 && x > 0)
            x = 0;
          // going down
          if (e.DeltaManipulation.Translation.Y > 0 && y > 0)
            y = 0;

          // Apply the computed value to the transform
          transform.X = x;
          transform.Y = y;
        }
      }
    }
```

So now the Image is zooming and moving as intended, let's add a bit of oomph to the manipulation.
This is made really easy with what the framework gives us. Using e.IsInertial and e.FinalVelocities is pretty much cheating. We just have to ensure we're not flinging the image out of bounds.

```csharp
    private void photo_ManipulationCompleted(object sender, ManipulationCompletedEventArgs e)
    {
      if (e.IsInertial)
      {
        Image photo = sender as Image;
        // Compute the inertial distance to travel
        double dx = e.FinalVelocities.LinearVelocity.X / 10.0;
        double dy = e.FinalVelocities.LinearVelocity.Y / 10.0;
        var transformgroup = photo.RenderTransform as TransformGroup;
        var transform = transformgroup.Children.First(c => c is TranslateTransform) as TranslateTransform;
        if (transform != null)
        {
          double x = transform.X + dx;
          double y = transform.Y + dy;

          // going left
          if (dx < 0)
          {
            if (Application.Current.Host.Content.ActualWidth - photo.ActualWidth * ScaleTransform.ScaleX > 0) return;
            if (x * ScaleTransform.ScaleX < Application.Current.Host.Content.ActualWidth - photo.ActualWidth * ScaleTransform.ScaleX)
              x = (Application.Current.Host.Content.ActualWidth / ScaleTransform.ScaleX) - photo.ActualWidth;
          }
          // going up         
          if (dy < 0)
          {
            if ((Application.Current.Host.Content.ActualHeight - 206) - photo.ActualHeight * ScaleTransform.ScaleX > 0) return;
            if (y * ScaleTransform.ScaleX < (Application.Current.Host.Content.ActualHeight - 206) - photo.ActualHeight * ScaleTransform.ScaleX)
              y = ((Application.Current.Host.Content.ActualHeight - 206) / ScaleTransform.ScaleX) - photo.ActualHeight;
          }
          // going right
          if (dx > 0 && x > 0)
            x = 0;
          // going down
          if (dy > 0 && y > 0)
            y = 0;

          // Apply the computed value to the animation
          PanAnimationX.To = x;
          PanAnimationY.To = y;

          // Trigger the animation
          Pan.Begin();
        }
      }
    }
```

And that's it. Now, this is far(!) from being production code, and there a few bugs here and there, but it was cool playing around with these gestures.

Next step would be to implement that nice bounce when you reach the edges of the screen. Hmmm...
