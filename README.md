# zShadows

A simple css library for generating shadows from a point light source

## Requirements

The target CSS engine must support var() and calc()

## Usage

Apply the .zBoxShadow style to any element you want to have its box shadow automatically calculated. Similarly, use the .zTextShadow style to calculate its text shadow.

## Examples

### A simple card
```css
.container {
	display: flex;
	justify-content: center;
	align-items: center;
	width: 400px;
	height: 400px;
}

.card {
	width: 50%;
	height: 67%;
	/*The z distance between this element and its parent*/
	--element-altitude: 1;
}
```

```html
<div style="container">
	<div style="card zBoxShadow"></div>
</div>
```

### Animate the card's elevation

```js
let start;

function update(timestamp) {
	if(start === undefined) {
		start = timestamp;
	}

	const elapsed = timestamp - start;
	document.getElementById("card").style.setProperty("--element-altitude") = Math.sin(elapsed % (2 * Math.PI)) + 1;

	window.requestAnimationFrame(update);
}

window.requestAnimationFrame(update);
```

### Make the light source move

```js
let start;

function update(timestamp) {
	if(start === undefined) {
		start = timestamp;
	}

	const elapsed = timestamp - start;
	const amplitude = 50;
	const elevation = 100;
	//Could also set --light-elevation on the document root to apply change to all elements that haven't overridden the property
	document.getElementById("card").style.setProperty("--light-elevation", Math.cos(elapsed % (2 * Math.PI)) * amplitude / 2 + elevation);

	window.requestAnimationFrame(update);
}

window.requestAnimationFrame(update);
```

### Add text shadows

```css
.card {
	display: flex;
	justify-content: center;
	align-items: center;
	width: 300px;
	height: 400px;
	--element-altitude: 1;
}

.text {
	--element-altitude: 2;
	--parent-elevation: 1;
}
```

```html
<div id="card" style="card zBoxShadow">
	<h2 id="text" style="text zTextShadow">Hey now, you're an all-star</h2>
</div>
```

### Animate the elements' elevation

```js
let start;

function update(timestamp) {
	if(start === undefined) {
		start = timestamp;
	}

	const time = (timestamp - start) / 1000;
	const cycle = time % (2 * Math.PI);
	const sin = Math.sin(cycle);
	const cos = Math.cos(cycle);
	const textStartElevation = 6;
	const textAmplitude = 3;
	
	const card = document.getElementById("card");
	const text = document.getElementById("text");
	//Assuming the card's parent is the document body with elevation = 0, the card's elevation is the same as its amplitude
	const cardElevation = sin + 1;
	const textAltitude = cos * textAmplitude + textStartElevation;

	card.style.setProperty("--element-altitude", cardElevation);
	text.style.setProperty("--parent-elevation", cardElevation);
	text.style.setProperty("--element-altitude", textAltitude);

	window.requestAnimationFrame(update);
}

window.requestAnimationFrame(update);
```

### Independent light sources for box and text shadows

```css
.card {
	--element-altitude: 2;
	--light-saturation: 33%;
}

.text {
	--parent-elevation: 2;
	--element-altitude: 5;
	--light-hue: 120deg;
}
```

```js
let start;

function update(timestamp) {
	if(start === undefined) {
		start = timestamp;
	}

	const time = (timestamp - start) / 1000;
	const rotation = time % (2 * Math.PI);
	const sin = Math.sin(rotation);
	const cos = Math.cos(rotation);
	
	const center = {
		x: window.innerWidth / 2,
		y: window.innerHeight / 2
	};
	
	const radius = (center.x ** 2 + center.y ** 2) ** 0.5;

	const card = document.getElementById("card");
	const text = document.getElementById("text");
	
	card.style.setProperty("--light-position-x", cos * radius + center.x + "px");
	card.style.setProperty("--light-position-y", sin * radius + center.y + "px");
	text.style.setProperty("--light-position-x", -cos * radius + center.x + "px");
	text.style.setProperty("--light-position-y", -sin * radius + center.y + "px");

	window.requestAnimationFrame(update);
}

window.requestAnimationFrame(update);
```

## Limitations

Currently only supports a single light source and shadow per element

Currently does not support negative elevations

Currently does not support inset box shadows

Shadows crossing element boundaries don't respect elevation changes. If the edges of your shaded elements are too close, it can look like the shadow is hanging in mid-air.

Elements lack a true z dimension, so there's no easy way to produce shadows for extruded shapes. However, it's probably doable with Houdini.

## Tips

In general, you'll always want to set each child element's --parent-elevation equal to its parent's (--parent-elevation + --element-altitude).

To keep pointer events working according to [Material Design](https://material.io/design) guidelines, you'll also want to assign each element's z-index to its --parent-elevation + --element-altitude.

By default, shadow angle is calculated between the light source's position and the viewport center point. This is a useful global approximation, but it means that all elements will have the same shadow angle.

If you want elements to have shadow angles relative to their own position and dimensions, you'll need to set each shaded element's --intersection-x and --intersection-y to some point on the surface. I'd recommend using either the element's center point, the farthest point on the element from the light source, or the midpoint between the two.