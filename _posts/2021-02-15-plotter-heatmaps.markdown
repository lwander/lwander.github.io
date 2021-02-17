---
layout: post
title: "Pen Plotter Heatmaps"
date: 2021-02-15 00:00:00 -0400
categories: plots
---

<a aria-describedby="footnote-label" href="#plotter">Pen plots</a> can fail
for a variety of reasons: a pen can run out of ink, paper can be misaligned,
the <a aria-describedby="footnote-label" href="#pen-lift-servo">servo</a>
lifting the pen can break... but the most nailbiting is watching the plotter
repeatedly cover the same spot in ink until the page tears!

<figure>
  <img src="/assets/images/heatmaps/failed-plot.webp" alt="Plot with torn paper">
  <figcaption>Circled in red you can see where the paper tore while plotting
  this pattern. Bits of ink-laden paper trail the pen's path leaving the tear;
  the result is an ugly plot!</figcaption>
</figure>

It's not easy to know when a plot will wind up tearing your paper. In fact,
looking at the <a aria-describedby="footnote-label"
href="rendered-preview">rendered preview</a> for the above plot shows a lot of
closely-drawn lines, but only one spot on the page actually winds up tearing:

<figure>
  <img src="/assets/images/heatmaps/source-plot.webp" alt="Rendered plot showing where paper tore">
  <figcaption>Circled in red is the spot that winds up tearing. But why only
  this spot? There are many places on the page that are completely
  black from the many lines that are rendered, but they don't tear.</figcaption>
</figure>

As you can see, looking at the rendered plot is not enough to know
if it will tear or not. We'll need some extra tools to point out these
problematic areas.

# Finding hotspots

The first step to solving this problem is simply knowing how many lines are
being drawn over the same point. If we can pinpoint "hotspots" on the plot, we
can guess where a plot will tear.

Since the tip of the pen has a non-zero width, it's not as simple as
calculating where the most line segments intersect -- we also need to account
for line segments that are close to one another as ink can bleed between
strokes.

An imperfect, but totally workable solution is to divide our plot into <a
aria-describedby="footnote-label" href="#square-size">1x1mm squares</a>, and
then record the squares each line segment covers.
See an example for a single line segment:

<figure>
  <img src="/assets/images/heatmaps/bresenham.svg" alt="Bresenham line drawing algorithm visualized">
  <figcaption>Depicted: <a
    href="https://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm">Bresenham's
    line drawing algorithm</a>. Image credit:
    <a href="https://commons.wikimedia.org/wiki/User:Crotalus_horridus">Crotalus
    Horridus</a>.
  </figcaption>
</figure>

This approach of finding the "squares" (i.e. pixels) that a line crosses
over is similar to what your computer does to render lines onto your computer 
screen.  There's [quite a few
approaches](https://en.wikipedia.org/wiki/Line_drawing_algorithm), each
with tradeoffs in accuracy, speed, and simplicity. I found [Bresenham's
algorithm](https://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm) works
just fine for my purposes.

# Enter the heatmap

Once we know how often each individual 1x1mm square is drawn over, we can turn
that information into a [heatmap](https://en.wikipedia.org/wiki/Heat_map).
Rendering the same plot from above using this approach shows us exactly where
the plot tore!

<figure>
  <img src="/assets/images/heatmaps/source-heatmap.webp" alt="Heatmap showing point where paper will tear">
  <figcaption>The color scale goes from blue (no lines) to red (most lines).
  In this case, red corresponds to 74 lines in a single 1x1mm square (see legend
  in the bottom left). Notice it lines up exactly with where the plot tore!
  </figcaption>
</figure>

# Possible next steps

Having these heatmaps has been enough to avoid surprise page tears, but there
are a few ways to take this further that I haven't explored yet:

## Automatically preventing page tears 

We could use this heatmap approach to automatically remove line
segments in very crowded parts of the plot. I don't know how easy this will be
to do without introducing unintended artifacts, though.

## Experimenting with papers, pens, inks, etc...

It could be interesting to run some experiments to find out exactly when certain
kinds of paper will tear. I have a hunch that the ink "wetness" on the page
plays a role, meaning factors like ambient humidity, and the time elapsed
between pen strokes over a crowded area wind up factoring into this. These are
quite a few variables to control, and will likely make getting accurate
measurements tricky.

## More "accurate" line drawing

Bresenham's algorithm does not perform any
[anti-aliasing](https://en.wikipedia.org/wiki/Spatial_anti-aliasing), and as a
result we might be misattributing how much a given square is covered by a given
line. However, I haven't needed this added accuracy so far.

# Lessons Learned

In practice, I've found that any more than 20 lines in a given 1x1mm square
will start to damage most Bristol paper, and over 40 lines is asking for a tear.
These numbers will probably vary from setup-to-setup, so I'll share what I've
been drawing with:

* TWSBI Eco Fountain Pens with an EF nib
* Strathmore 300 Series Bristol Vellum Paper
* Pilot Fountain Pen Ink

Happy plotting!

<footer>
  <ol>
	<li id="plotter">
	  <a href="https://en.wikipedia.org/wiki/Plotter">Wikipedia background</a>,
      (non-affiliate) link to plotter I use: <a
      href="https://axidraw.com/">https://axidraw.com/</a>.
	</li>
	<li id="pen-lift-servo">
      This actually happens quite a bit on my Axidraw! I recommend keeping a few
      spares handy if you're using this machine.
	</li>
	<li id="rendered-preview">
      I render my plots with a little Javascript + HTML canvas.
    </li>
	<li id="square-size">
      1x1mm is semi-arbitrary, but has provided good results so far. It's
      intentionally a little wider than the pen tips I use to account for ink
      bleedage. Increasing the
      resolution doesn't provide much extra info, and just slows down rendering
      of the heatmap.
    </li>
  </ol>
</footer>
