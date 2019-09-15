---
layout: post
title:      "Language  Flashcards or How I Learned CSS Animations and Love the Fetch API"
date:       2019-09-15 21:12:01 +0000
permalink:  language_flashcards_or_how_i_learned_css_animations_and_love_the_fetch_api
---


I've always loved languages.  The idea of meeting people whose lives and experiences can be vastly different from yours or unexpectedly similar, and having an immediate connection and rapport is wonderful.  It can be great fun to see someone's face light up when they, with noticeable accent, say "Thank you" when you hold a door open and you respond in their language "Walang anuman!"

When the Sinatra project time came, it seemed a perfect fit to make a language flashcard app.  A user can have many decks.  A deck can have many cards, a card belongs to a deck and a deck to a user and so on.  While the idea fit very well with the requirements, there were challenges in making the app function the way I wanted it to.  The behavior of the flashcards was the source of  many of those.

Firstly, the flashcards cannot really be called flashcards if they don’t have a front and a back and can flip between the two.  Otherwise it’s maybe a quiz page or a dictionary?  Neither bad ideas, but not what I wanted.  Secondly, the flashcards need to vary.  It’s too easy to learn the order of the answers and miss learning the real content, i.e. the translation between the two languages.  In addition to it varying the order, there also needs to be an end.  Just randomly drawing a new card could work, but I like the idea of getting all the way through a deck of cards in a study session.

Let’s look at the flip effect first.  CSS animation makes this pretty easy.  To start with, you’ll need a container that will set the width and height of the card.  Essentially, this container holds all the stuff to make the card and, using CSS, you tell it what stuff should be seen and when.  The HTML for my cards looks like this:

```
<div class="card-area" id="card-area">
		<i class="far fa-check" id="checkmark"></i>
		<i class="far fa-times" id="big-x"></i>
		<div class="card" id="card" >
			<div class="card-face card-front" id="front">
			</div>
			<div class="card-face card-back" id="back">
				<p id="question"></p>
				<p id="correct-answer"></p>
			</div>
		</div>
	</div>
</div>
```

The container is .card-area.  You really only need to divs within the container (i.e. front and back), but I opted to also show a “right” and “wrong” indicator (#checkmard and #big-x, respectively).  The front and back are then contained within the #card div.  The CSS for my .card-area is:

```
.card-area {
	width: 260px;
	height: 300px;
	perspective: 800px;
	display: flex;
	justify-content: center;
	align-items: center;
}
```

This sets the size of the cards, centers the content, and, via perspective, gives the cards a cool  3-d appearance when the flip animation occurs.

The front and back are largely where all the special CSS goes and where the animation is defined.  The CSS for these are broken into three sections, .card-face – where the CSS properties shared between front and back go – and .card-front and .card-back – where the CSS specific to only one of the two  resides.  Only the back in my case had CSS specific to it, however.  Here’s the CSS for these sections:

```
.card-face {
	align-items: center;
	backface-visibility: hidden;
	background: none;
	border: 1px solid red;
	border-radius: 15px;
	box-shadow: 2px 2px 2px 1px red;
	display: flex;
	height: 100%;
	justify-content: center;
	position: absolute;
	width: 100%;
}
```
```
.card-back {
	flex-direction: column;
	transform: rotateY( 180deg );
}
```

With the .card-front CSS empty.  In .card-face, the only bit related to the flip animation is `backface-visibility: hidden;`.  The backface property indicates whether a mirror image of the front side content should be  show when the “card” is flipped.  Think of writing on a piece of glass and turning it over.  As I want to have a distinct front and back, I do not want the “downward” facing side visible.  In .card-back, the transform is defined.  Just as would be expected in the physical world, its a rotation of the card over 180 degrees.

The last CSS is applied to a class that is added when the card flip is activated (via JavaScript on a button click in my case).

```
.card.is-flipped {
	transform: rotateY( 180deg );
}
```
So, when the card has the .is-flipped class, the animation happens.

As I said, the animation is triggered by a button click event in JavaScript.  It is in this .js file that the second requirement for a flashcard app takes place—the randomization and beginning to end behavior.  Once I, with the help of my adviser, figured this out, it was actually deceptively easy and straight forward.  Two parts are needed, a route in Sinatra to handle the request, and a call to the Fetch api to make the request.  As I only (for now) need to get data from  the server, a get request sufficed.

The Fetch api is pretty easy to use with minimal understanding (you don’t need to have an in depth knowledge of how promises work) and some copying and pasting skill.  The fetch() method can take two parameters, resource—which is required and inti—which is optional.  The resources is the path or URI to which the request should be made.  I won’t go into the init parameter as it wasn’t needed here; the settings in the init object were either unneeded here or only the default behavior was.  The resource matches the route where the needed data will be served.  In my case it was `get ‘/decks/cards’` for the route and a corresponding fetch of `fetch(‘/decks/cards’`.  The cards that belonged to the deck to be studied were queried with ActiveRecord and the subsequent array was `.shuffle` ed and returned as a JSON object.  That object was parsed and each card could then be displayed one at a time and flipped through.

This project provided quite a few interesting (and at times, frustrating) challenges.  The kind of problem solving required to solve these challenges is exactly what makes programming fun.  It’s like each error is a  clue in a treasure hunt.  Each time you resolve an error you’re rewarded with a new one to work through, moving closer and closer to the X that marks the spot (or minimally acceptable app that will meet the requirements and actually load and behave when served).
