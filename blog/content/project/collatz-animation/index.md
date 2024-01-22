---
title: Animating the Collatz Conjecture
date: '2024-01-22T00:00:00Z'
math: true
draft: False
links:
  - icon: github
    icon_pack: fab
    name: Code
    url: https://github.com/AryamanReddi99/collatz-visualisation
tags:
  - Math
image:
  placement: 1
  caption: 'Image credit: **Aryaman Reddi**'
summary: Animating one of the most beautiful and confounding problems in math.
---
## The Collatz Conjecture
The Collatz conjecture is among the strangest, simplest, and most beautiful problems in math.

The conjecture states the following:

{{< rawhtml >}}
<div style="border: solid;">
  {{< math >}}
  $$
  \text{For an arbitrary positive integer n, consider the function f:}
  $$
  {{< /math >}}

  {{< math >}}
  $$
  f(n) = \begin{cases}n/2 & \text{if }n\%2=0 \text{ (even)}, \\
  3n+1 & \text{if }n\%2=1 \text{ (odd)}\end{cases}
  $$
  {{< /math >}}

  \begin{align}
    &\text{By starting with any positive integer and repeatedly} \\
    &\text{performing this operation to the output, an integer} \\
    &\text{sequence is generated which will always eventually reach 1.}
  \end{align}
</div>
<br>
{{< /rawhtml >}}

Let's try an arbitrary number (e.g. 11) to see what happens when we repeatedly apply the rule.

11 → 34 → 17 → 52 → 26 → 13 → 40 → 20 → 10 → 5 → 16 → 8 → 4 → 2 → 1 → 4 → 2 → 1 → 4 → 2 → 1 . . .

When we reach 1, we get stuck in a loop (1 → 4 → 2 → 1) that prevents us from visiting any other numbers henceforth. This is the heart of the Collatz conjecture: **all sequences eventually get stuck at 1.**

In mathematics, a conjecture refers to a proposition that is presented without formal proof. Despite its simplicity, a proof of the Collatz conjecture has evaded mathematicians thus far. Empirically, it has held true for every number we've checked so far ([as of 2020, we've verified all integers from 1 to 2^68](https://link.springer.com/article/10.1007/s11227-020-03368-x)), but we would only need 1 counterexample to disprove it.

On this page, we'll see how to animate the Collatz conjecture using the program **Processing**.

{{< rawhtml >}}
  <div class="clearfix">
    <div class="img-container-2">
    <img src="videos/collatz_parallel_purple.gif" style="margin-bottom: 10px">
    <figcaption>angle = 0.15</figcaption>
    </div>
    <div class="img-container-2">
    <img src="videos/collatz_parallel_green.gif" style="margin-bottom: 10px">
    <figcaption>angle = $\pi/2$</figcaption>
    </div>
  </div>
{{< /rawhtml >}}
{{< figure src="videos/collatz_parallel_autumn.gif" caption="angle = $\pi/3$" width="50%">}}

As far as I know, the idea to illustrate the Collatz conjecture with these branching patterns comes from the book **Visions of Numberland** by Edmund Harriss and Alex Bellos.

This [Numberphile video](https://www.youtube.com/watch?v=LqKpkdRRLZw&pp=ygUTY29sbGF0eiBjb25qZWN0dXJlIA%3D%3D) explains a bit about how these branching images are created using the conjecture.

These animations are based on [this video by The Coding Train](https://www.youtube.com/watch?v=EYLWxwo1Ed8&t=1234s) which explains how to generate a static image of the Collatz tree. Start here if you've never used processing before.

You can find the code for this project [here.](https://github.com/AryamanReddi99/collatz-visualisation)

---

## Code

We need to start with downloading Processing, a Java-based program which you can find [here](https://processing.org/).

The basic idea is that we'll first generate complete sequences of integers from repeatedly applying the Collatz function to a range of seed numbers until they inevitably reach 1. These will be the 'branches' of our tree. 

At each iteration of the animation (every call of the 'draw' function), we'll extend each of these branches by 1 step from their end point (1). 

Every time we extend a branch, we'll apply a simple rule: If the number we're going to is even, we'll rotate the branch by some angle clockwise. If it's odd, we'll rotate the branch by that same angle anticlockwise. 

We'll continue doing this until all branches have reached their seed number.

Let's define some variables:

```java
//Global
int i = 0; // animation iteration
int max_len = 0; // longest sequence length
ArrayList<IntList> sequences = new ArrayList<IntList>();
ArrayList<Integer> sequence_colours = new ArrayList<Integer>();

//Aesthetics
int window_width = 1080;
int window_height = 1080;
float origin_x = 0.2;
float origin_y = 0.5;
int scaling = 1; // scale down animations in case your monitor is too small
float line_len = 10/scaling; // length between numbers on a branch
int line_alpha = 50;
float angle = 0.15; // angle between numbers on a branch
int num_sequences = 20000; // first 20000 sequences
```

We'll then set the window size for our animation:

```java
void settings() {
  size(window_width/scaling, window_height/scaling);
}
```

We'll need a function to calculate each next step in the sequence:

```java
int collatz(int n) {
  if (n%2 == 0) { // even
    return n/2;
  } else { // odd
    return (n*3+1)/2;
  }
}
```

And a function which creates a sequence based on a seed number:

```java
IntList collatz_thread(int n) { 
  // return collatz sequence spawned by n until 1 is reached
  IntList sequence = new IntList();
  sequence.append(n);
  while (n!=1) {
    n = collatz(n);
    sequence.append(n);
  }
  return sequence;
}
```

We'll then specify the ``setupproc()`` function, which runs before the drawing starts in Processing.

Here, we'll generate the sequences starting from 1 to our chosen highest seed and assign each sequence a colour (this can be random or follow some specific rules).

```java
void setup() {
  background(10);
  strokeWeight(2);
  for (int seed=1; seed<num_sequences+1; seed++) {
    IntList sequence = collatz_thread(seed); // generate collatz sequence
    sequence.reverse();
    sequences.add(sequence);
    if (sequence.size() > max_len) {
      max_len = sequence.size();
    }
    
    // Custom colours
    int final_val = sequence.get(sequence.size()-1);
    int sequence_colour;
    if (final_val%2==1) {
      sequence_colour = #FF0D00; //red
    } else {
      sequence_colour = #08FF05; //green
    }
    
    // Random colours
    //int r = int(random(0,255));
    //int g = int(random(0,255));
    //int b = int(random(0,255));
    //int sequence_colour = dec_to_colour(r, g, b);
    
    sequence_colours.add(sequence_colour);
    
  }
  println("Animating", sequences.size(), "sequences");
}

int dec_to_colour(int r, int g, int b){
  return color(r,g,b);
}
```

Finally, we can write the ``draw()`` function. In Processing, ``draw()`` is repeatedly invoked until the program is stopped or ``noLoop()`` is called. Using this, each call of ``draw()`` will extend the branches of our Collatz tree.

In ``draw()``, we start by placing the origin point at a convenient place on our canvas.

We then extend each branch of each sequence by 1 number. To do this, we first traverse along each branch until we reach the last 'leaf' that was drawn in the previous call of ``draw()`` (we must invoke ``noStroke()`` while doing this so we don't draw over anything). 

We then create one more line extending from the end of this branch (of a specific angle and colour) using the ``line()`` function, which draws directly on our canvas.

If you want to save these frames to create an animation like those shown on this page, you can uncomment the 2 lines after ``// Video`` which will save the individual frames to a folder called 'frames'.

You can then use the **Movie Maker** tool in Processing to render these frames as a video (explained [here](https://www.youtube.com/watch?v=G2hI9XL6oyk&t=376s&pp=ygUWbW92aWUgbWFrZXIgcHJvY2Vzc2luZw%3D%3D)).

Finally, once all branches have been completed, we stop the animation.

```java
void draw() { 
  // each time draw() is called, one iteration of the sequence is animated, i.e.
  // each collatz thread is extended by one step
  for (int sequence_idx=0; sequence_idx<sequences.size()-1; sequence_idx++) {
    resetMatrix(); // returns drawing tool to origin
    translate(origin_x*width, origin_y*height); // move start of each thread to the same location
    rotate(PI/2); // start angle rotation
    
    IntList sequence = sequences.get(sequence_idx);
    if (sequence.size() > i) { // reach end position of sequence drawn so far
      noStroke();
      for (int j=0; j<i; j++) {
        int val = sequence.get(j);
        val_rotation(val);
        translate(0, -line_len);
      }
      int final_val = sequence.get(i);
      val_rotation(final_val);
      stroke(sequence_colours.get(sequence_idx), line_alpha);
      line(0, 0, 0, -line_len); // draw 'vertical' line
    }
  }
  // Video
  //println("Frame", i, "saved");
  //saveFrame("frames/####.png");
  
  i++;
  if (i>=max_len) {
    noLoop();
    println("Finished");
  }
}
```

---

## Notes

To try this project for yourself, you can just download the code [here](https://github.com/AryamanReddi99/collatz-visualisation) and run it in processing.

The file `collatz_branches_parallel/collatz_prog.pde` will animate the sequences in parallel (all the branches at once) whereas the file `collatz_branches_serial/collatz_sketch.pde` will animate them one after another.

It should be noted that the `angle` variable has a huge effect on the patterns that are generated, as certain 'frequencies' can result in fractal-like structures (try various divisions of $\pi$ to see what I mean!).

