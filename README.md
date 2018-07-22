# Android additive animations

Additive animations for Android!
An easy way to additively animate a huge number of properties of all kinds of objects, with convenient builder methods for `View`s.

Get a good overview of this library here: https://medium.com/@david.gansterd/bringing-smooth-animation-transitions-to-android-88786347e512


# Quick start
Here is a sample of what additive animations can do to the user experience (note: there seem to be a few dropped frames in the gif which aren't present when running on a device):


![Additive animations demo](https://github.com/davidganster/android_additive_animations/blob/master/gif/single_view.gif?raw=true)


The amount code required to produce this animation is trivial:

```java
public boolean onTouch(View v, MotionEvent event) {
    AdditiveAnimator.animate(animatedView).x(event.getX()).y(event.getY()).setDuration(1000).start();
    return true;
}
```

Additionally, `AdditiveAnimator` supports animating multiple targets simultaneously without any boilerplate:

```java
new AdditiveAnimator().setDuration(1000)
                      .target(myView1).x(100).y(100)
                      .target(myView2).xBy(20).yBy(20)
                      .start();
```
**New in 1.6:**

1.6 added a some convenience features, such as the ability to switch duration midway to building an animation, providing a `SpringInterpolator` class, and being able to switch back to the default interpolator using the `switchToDefaultInterpolator()` method.

Then main attraction of 1.6 though:

You can now animate the same property for multiple views without looping.

```java
new AdditiveAnimator().targets(myView1, myView2).alpha(0).start();
```

To achieve a delay between the start of the animation of each target, you can optionally add the 'stagger' parameter to add a delay between each of the animations.
```java
long staggerBetweenAnimations = 50L;
new AdditiveAnimator().targets(Arrays.asList(myView1, myView2), staggerBetweenAnimations).alpha(0).start();
```
In this example, `myView1` is faded out 50 milliseconds before `myView2`.

Starting with 1.6.1, the delay between the animation of the views is preserved when using `then()` chaining:
```java
long staggerBetweenAnimations = 50L;
AdditiveAnimator.animate(Arrays.asList(myView1, myView2), staggerBetweenAnimations).translationYBy(50).thenWithDelay(20).translationYBy(-50).start();
```

The timeline of this animation looks like this:
`myView1` is translated by 50 pixels at delay 0.
`myView2` is translated by 50 pixels at delay 50.
`myView1` is translated by -50 pixles at delay 20.
`myView2` is translated by -50 pixles at delay 70.

Check out `MultipleViewsAnimationDemoFragment` in the demo app for an example of this!

# Animating all kinds of objects and properties
In addition to the builder methods for views, there are multiple options for animating custom properties of any object.
The first option is subclassing `BaseAdditiveAnimator` and providing your own builder methods (which are usually one-liners) such as this:

```java
class PaintAdditiveAnimator extends BaseAdditiveAnimator<PaintAdditiveAnimator, Paint> {
    private static final String COLOR_ANIMATION_KEY = "ANIMATION_KEY";

    // Support animation chaining by providing a construction method:
    @Override protected PaintAdditiveAnimator newInstance() { return new PaintAdditiveAnimator(); }

    // Custom builder method for animating the color of a Paint:
    public PaintAdditiveAnimator color(int color) {
        return animate(new AdditiveAnimation<>(
            mCurrentTarget, // animated object (usually this is the current target)
            COLOR_ANIMATION_KEY, // key to identify the animation
            mCurrentTarget.getColor(), // start value
            color)); // target value
    }

    // Applying the changed properties when they don't have a Property wrapper:
    @Override protected void applyCustomProperties(Map<String, Float> tempProperties, Paint target) {
        if(tempProperties.containsKey(COLOR_ANIMATION_KEY)) {
            target.setColor(tempProperties.get(COLOR_ANIMATION_KEY).intValue());
        }
    }
}
```

The second option is to simply provide a `Property` for the object you want to animate, plus (if needed) a way to trigger a redraw of your custom object:

```java
// Declaring an animatable property:
FloatProperty<Paint> mPaintColorProperty = new FloatProperty<Paint>("PaintColor") {
    @Override
    public Float get(Paint paint) { return Float.valueOf(paint.getColor()); }

    @Override
    public void set(Paint object, Float value) { object.setColor(value.intValue()); }
};

...

// Using the property to animate the color of a paint:
AdditiveObjectAnimator.animate(myPaint)
    .property(targetColor, // target value
              new ColorEvaluator(), // custom evaluator for colors
              mPaintColorProperty) // how to get/set the property value
    .setAnimationApplier(new ViewAnimationApplier(myView)) // tells the generic AdditiveObjectAnimator how to apply the changed values
    .start();
```

A more complete example of both of these approaches can be found in the sample app in  `CustomDrawingFragment.java`.

Both versions don't require a lot of code, and the few lines you have to write are almost always trivial.
# Integration
To use `AdditiveAnimator` in your project, add the following lines to your `build.gradle`:
```
dependencies {
    compile 'at.wirecube:additive_animations:1.6.2'
}
...
repositories {
    jcenter()
}
```

**Note**: There is a  breaking change when migrating from a version <1.5.0 to a version >= 1.5.0:
Instead of subclassing `AdditiveAnimator`, you now have to subclass `SubclassableAdditiveViewAnimator` instead.
Sorry for the change, it was necessary due to Java constraints (nesting of generics across subclasses) and improves interop with Kotlin (no more generic arguments required!).

**Note**: There is another breaking change when migrating from <1.6.0 to >= 1.6.0:
You have to implement a new abstract method (`getCurrentPropertyValue()`) when subclassing `BaseAdditiveAnimator`.
This method is only called when using tag-based animations, instead of property-based ones. If your subclass does not use tag-based animations, you can simply  `return null;`.

# License
`AdditiveAnimator` is licensed under the Apache v2 license:

```
Copyright 2017 David Ganster

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
