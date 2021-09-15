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

	--element-altitude: 1;
}
```

```html
<div style="container">
	<div id="card" style="card zBoxShadow"></div>
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
	document.getElementById("card").style["--element-altitude"] = Math.sin(elapsed) + 1;
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
	document.documentElement.style["--light-x"] = 
	document.getElementById("card").style["--element-altitude"] = Math.sin(elapsed) + 1;
}

window.requestAnimationFrame(update);
```

### Add text shadows

```css
.card {
	display: flex;
	justify-content: center;
	align-items: center;
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

### Animate the text's elevation

```js
let start;

function update(timestamp) {
	if(start === undefined) {
		start = timestamp;
	}

	const elapsed = (timestamp - start) / 1000;
	const cardAltitude = Math.sin(elapsed) + 1;
	//Assuming card's parent is the document root with elevation = 0, the card's elevation is the same as its altitude
	const cardElevation = cardAltitude;
	
	document.getElementById("card").style["--element-altitude"] = cardAltitude;
	document.getElementById("text").style["--parent-elevation"] = cardAltitude;
	document.getElementById("text").style["--element-altitude"] = cardElevation + Math.cos(elapsed) + 1;
}

window.requestAnimationFrame(update);
```

### Independent light sources for box and text shadows

```js
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

While it'll look fine for simple use cases, if you apply the same shadow to an element's text and box, it'll look odd in more complex scenarios. What you really want is to raise the text above the element's surface.

I'd recommend always making new child element for shaded text and setting its altitude relative to its parent in the same way described above. It might also be worth making the child element's background transparent.

Basically, don't apply .zBoxShadow and .zTextShadow to the same element.