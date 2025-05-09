+++
title = 'SVGs: Drawing Simple Paths with Straight Lines'
date = 2025-05-09T22:35:30+02:00
tags = ["svg"]
technologies = []
+++

I like the SVG vector graphic format.
The coolest thing about vector graphics is that we don’t need to worry about image quality, regardless of the display size.
From a web development perspective, this is a huge benefit for responsive layouts.
Who knows on what weird device your user is viewing the page?
With pixel-based images, you can either create high-quality images that explode in size, or accept that on particularly large screens your image might appear pixelated.

With vectors, you simply store paths that are then rendered on the screen.
They scale to any size imaginable.
I love using SVGs for that purpose but have never really understood how they work.
Now, I’m trying to change that.

## Format

An SVG file is actually just HTML syntax.
This means we can "code" our images if we want to.
We can also use code to manipulate the HTML and generate or change an image programmatically.
Think of the possibilities!
It’s totally feasible to use JavaScript to animate an SVG image, for example—
or to load image content in a meaningful order... cool stuff.

The basic layout looks like this:
We have an opening and closing `<svg></svg>` HTML tag.
An `<svg>` tag has `height` and `width` attributes to specify the size of the image.
Within that tag, we can include one or more paths.
A path is a collection of coordinates that define the shape.
It can have a `stroke` color and a `fill` color.

A simple SVG would look like this:

```html
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
    <path d="M 10 10 L 90 10" stroke="red" fill="none"/>
</svg>
```

Here, we have an SVG that is 100px wide and 100px tall.
It contains a path.
The path starts by moving the current point to `x=10` and `y=10` using the "move" command `M`.
It then draws a straight line `L` from the current location to `x=90` and `y=10`.
So, we simply move 80px to the right.
We specify a stroke color of `red` and no fill color.
On a single line, we don’t actually have any area to fill.
The image looks like this:

![horizontal line](horizontal-line.svg)

Now that we know how to draw a line, we can use it to draw a rectangle.
We do that by continuing the path with straight lines (`L`) from the current location to the next coordinates:

```html
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
    <path d="M 10 10 L 90 10 L 90 90 L 10 90 L 10 10" stroke="red" fill="none"/>
</svg>
```

That will look like this:

![square](square.svg)

### Closing the Path

In the previous example, we completed the square by manually moving back to the initial position at `x=10, y=10`.
But we can actually do that automatically with the `Z` command, which closes the path by returning to the starting point.

```html
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
    <path d="M 10 10 L 90 10 L 90 90 L 10 90 Z" stroke="red" fill="none"/>
</svg>
```

Notice how the final `L 10 10` was simply replaced with `Z`.

### Fill Colors

Now that we actually have a closed shape, we can define a fill color.
Any valid HTML color specification should work here (e.g., names, hex codes, RGB, etc.).

```html
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
    <path d="M 10 10 L 90 10 L 90 90 L 10 90 Z" stroke="red" fill="#ddd"/>
</svg>
```

![square fill](square-fill.svg)

If we don’t close the path by returning to the initial position, the fill will still assume the shape that the `Z` command would have created, but it won’t add the final stroke:

```html
<svg width="100" height="100" xmlns="http://www.w3.org/2000/svg">
    <path d="M 10 10 L 90 10 L 90 90 L 10 90" stroke="red" fill="#ddd"/>
</svg>
```

![square fill open](square-fill-open.svg)

Notice the missing `Z`, and that in the final shape, there’s no stroke closing the square.
Still, the fill shape is defined as if the path had been closed.
