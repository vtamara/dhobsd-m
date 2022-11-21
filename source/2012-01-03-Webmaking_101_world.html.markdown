---
title: Webmaking_101_world
date: 2012-01-03 12:00 UTC
tags:
---
[http://fe.pasosdejesus.org/sites/FE.PASOSDEJESUS.ORG/spages/Organeta.jpg]

!Summary:

* Sections: <div>
* Sheet of Notebook <ul>. Each cell: <li>
* Desing on placemat: <img>
* Words on book (Nuestro Pan Diario): <blockquote>
* Change in font: <span>
* Timer: <time>
* Keyboard: <ol>

!Explanation:

* This photo has like 5 sections, each one could be a <div>, the upper one has a notebook, a placemat and a book, below the second one has information of the keyboard, the third one has buttons and also the fourth one; the fifth section has the keys.  See div reference at https://developer.mozilla.org/en/HTML/Element/div
* The sheet of a notebook that can be seen has several cells, since all of them are empty, we could think of them as unordered and represent them with an <ul> and each cell with a <li>.  See https://developer.mozilla.org/en/HTML/Element/ul
* The desing on the placemat that is below the keyboard can be represented with an <img> tag. See https://developer.mozilla.org/en/HTML/Element/img
* There is a book in the upper right side of the picture, with the words "Nuestro Pan Diario" these words look like a quotation and a little to the right of the margin of the book then we could think of <blockquote>, see https://developer.mozilla.org/en/HTML/Element/blockquote
* In the second section where there is information with different fonts, for example the word Yamaha is in a font family while !PortaSound is in another one.  Each one could be inside a <span> element, see https://developer.mozilla.org/en/HTML/Element/span.
* The timer is not showing the actual date neither a time, but it seems like a digital clock then we compare it with a <time> element, that allows to markup a date or time, see https://developer.mozilla.org/en/HTML/Element/time.
* Since the notes in a Piano are like an "ordered list of elements" we consider them <ol>, and each key like an item list <li>.  See reference at https://developer.mozilla.org/en/HTML/Element/ol

!A extra-minimal model

The previous description in HTML http://htmlpad.org/piano/ editable at http://htmlpad.org/piano/edit

