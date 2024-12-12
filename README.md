# Week 12: Classes

**What are we building:** We're going to build an animated flock of "birds", heavily influenced by [this example](https://p5js.org/examples/classes-and-objects-flocking/)

## What are Classes?

A class is a way of declaring a new "type" of variable. They're allowed to have `properties` (variables owned by each instance of the class) and `methods` (functions you can call on a class instance). They also have a `constructor`, which is the method that creates an instance of the class

```js
class Car {
    constructor(color, gasPercent) {
        this.color = color;
        this.gasPercent = gasPercent;
    }

    // This is a method that lets us know if this particular car has gas
    isTankEmpty() {
        return this.gasPercent == 0;
    }
}

let car1 = new Car('red', 100);
car1.isTankEmpty(); // False


let car2 = new Car('green', 0);
car2.isTankEmpty(); // True

```

## Step 0

Ok, so to start off, let's open a new p5 project:

> https://editor.p5js.org/ 

and we're going to do some of the initial scaffolding for you. In particular, we're going to define a class `Bird`, define an array for all the birds that exist, and make it so that when you click on the canvas, it creates a new bird.


```js
class Bird {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    
    this.hue = 128; // We'll change this later
    
    this.angle = 0; // We'll change this later
    
    this.size = 3; // We'll change this later
  }
  
  draw() {
    colorMode(HSB);
    fill(color(this.hue, 255, 255));
    stroke(255);
    
    push();
    translate(this.x, this.y);
    rotate(this.angle);
    beginShape();
    vertex(0, -this.size * 2);
    vertex(-this.size, this.size * 2);
    vertex(this.size, this.size * 2);
    endShape(CLOSE);
    pop();
  }
  
}

let allBirds = [];

let CANVAS_SIZE = 400;

function setup() {
  angleMode(DEGREES);
  createCanvas(CANVAS_SIZE, CANVAS_SIZE);
  createP('Drag the mouse to generate new birds.');
  
  // Randomly initialize a few birds
  for (let i = 0; i < 25; i++) {
    let randomX = random(CANVAS_SIZE);
    let randomY = random(CANVAS_SIZE);
    allBirds.push(new Bird(randomX, randomY))
  }
}

// On mouse drag, add a new bird
function mouseDragged() {
  allBirds.push(new Bird(mouseX, mouseY));
}

function draw() {
  background(0);
  
  for (let i = 0; i < allBirds.length; i++) {
    allBirds[i].draw();
  }
}

```

So with this setup, we get a few random "birds", and we can
click and drag the mouse to make more.

But:

* the birds all look the same
* the birds aren't moving at all

so let's change that

## Step 1

Let's update the constructor of our `Bird` class to take an extra argument for `hue`:

```js
// Your constructor should look like this:
constructor(x, y, myHue) {

  // and then you'll need to use myHue to set the value...
```

and then in the places where we create Bird, we'll pass in a new value to the `new Bird()` call.

> Challenge 1: Try making all the initial Birds have a hue of 0, and all the mouse-click birds have a hue of 128

> Challenge 2: Try making the hue depend on the x or y value of where the bird was created

## Step 2

Now, let's make the birds move in the direction they're facing.

To do that, we're going to make 2 main changes:

1) We'll add a method `updateLocation` to the Bird class
1) We'll call that method for every bird before we draw it

For the first part, let's add something like:

```js
updateLocation() {
    let speed = 1;

    this.x += cos(this.angle - 90) + speed;
    this.y += sin(this.angle - 90) + speed;

    this.x = constrain(this.x, 0, CANVAS_SIZE);
    this.y = constrain(this.y, 0, CANVAS_SIZE);
}
```

to the `Bird` class.

> Challenge 3: Call the `updateLocation()` method for each bird, before you call `draw()` for each bird

By this point, you should have a flock of birds that all fly in the same direction, and crash into the wall. Let's make this a bit more interesting

## Step 3

Let's set the `angle` value for each bird independently:

* Option 1: `random(360)` to set a random angle
* Option 2: set the angle based on some other aspect of the bird (e.g. the starting location, or hue)

By this point, you should have a bunch of birds flying in different directions, but they're still not changing directions...

## Step 4

Let's change the `angle` value, whenever the bird hits the edge of the canvas:

> Challenge 4: Update `updateLocation()` to randomly change the angle of the bird if the `x` or `y` values hit either `0` or `CANVAS_SIZE`

Alright, so now we have a bunch of birds that are randomly ping-ponging around the canvas, let's see if we can have them speed up or slow down...

## Step 5

In `updateLocation()`, let's (instead of using a constant speed of `1`) define a speed on each bird.

> Challenge 5: Update the `Bird` class to define its own speed, and use that when updating location

```js
this.speed = random(0.5, 2); // Something like this
```

But at this point, the birds are still all kind of ignoring each other, let's change that:

## Step 6

We'll update `updateLocation` to pass in the full list of birds:

```js
  updateLocation(allBirds) {
    
    let anyTooClose = false;
    for (let i = 0; i < allBirds.length; i++) {
      if (allBirds[i] == this) {
        continue;
      }
      
      let distance = sqrt((this.x - allBirds[i].x) * (this.x - allBirds[i].x) + (this.y - allBirds[i].y) * (this.y - allBirds[i].y));
      
      if (distance < this.size) {
        anyTooClose = true;
      }
    }

    // ... rest of method
```

and the `draw` method at the bottom of the file should now look like:

```js
function draw() {
  background(0);
  
  for (let i = 0; i < allBirds.length; i++) {
    allBirds[i].updateLocation(allBirds);
    allBirds[i].draw();
  }
}
```

> Challenge 6: Update the `updateLocation` method to have the birds react when they get too close to one another. You can have them stop entirely by changing `this.speed` to be `0`, or you could change the `angle` to have them fly away from each other

or if you wanted to have fun with it:

> Challenge 7: Try only having birds of the same `hue` stop when they collide, and birds of different `hues` instead fly away from each other (`abs(this.hue - allBirds[i].hue) < 10` is a hint on how to figure out if the birds are similar enough in hue)
