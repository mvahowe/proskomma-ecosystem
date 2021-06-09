.. _basics:

###############
Getting started
###############

This tutorial explains how to get up and running with Proskomma quickly.

We will import a short sample document and we will perform a series of queries on said document.

It is not the aim of this tutorial to provide a comprehensive overview of the ways in which Proskomma
reads, parses, stores, represents, and edits texts that after importing USFM or USX formats. Rather the 
aim is to show how you can query. To fully understand how Proskomma querying works, it will be necessary to 
explain the basics of the Proskomma data model.

In this tutorial you will create a javascript script called :code:`tutorial.js` that we will gradually fill 
with commands that are meaningful. 

We will be running this file using :code:`node tutorial.js`. 

This tutorial assumes that you have successfully installed Proskommma. If you have not, you could still run
:code:`npm init` and :code:`npm install proskomma`

Let's start by creating an empty file called `tutorial.js`

First we need to load proskommma, assuming you have followed the installation instructions
In the tutorial.js file, add the following line

.. code:: javascript

    const { Proskomma } = require('proskomma');


We now have access to the module in our script. 

Let's work with a short sample of the beginning of Genesis in unfoldingWord's ULT. 
The one change to this document is that the backslash is doubled (it is escaped) in comparison
to normal USFM. We did so in order to include it here as a sample, without having to worry about
reading a document from the file system. Do not be distracted by the 
double slash, your normal USFM documents do not need it. 
Add the following to `tutorial.js`


.. code:: javascript

    const sample = `\\id GEN EN_ULT en_English_ltr unfoldingWord Literal Text Thu Jul 25 2019 09:33:56 GMT-0400 (EDT) tc
    \\usfm 3.0
    \\ide UTF-8
    \\h Genesis
    \\toc1 The Book of Genesis
    \\toc2 Genesis
    \\toc3 Gen
    \\mt Genesis 

    \\ts\\*
    \\c 1
    \\p
    \\v 1 In the beginning, God created the heavens and the earth. 
    \\v 2 The earth was without form and empty. Darkness was upon the surface of the deep. The Spirit of God was moving above the surface of the waters.

    \\v 3 God said, “Let there be light,” and there was light.
    \\v 4 God saw the light, that it was good. He divided the light from the darkness.
    \\v 5 God called the light “day,” and the darkness he called “night.” This was evening and morning, the first day. 

    \\p
    \\v 6 God said, “Let there be an expanse between the waters, and let it divide the waters from the waters.”
    \\v 7 God made the expanse and divided the waters which were under the expanse from the waters which were above the expanse. It was so.
    \\v 8 God called the expanse “sky.” This was evening and morning, the second day.

    \\p
    \\v 9 God said, “Let the waters under the sky be gathered together to one place, and let the dry land appear.” It was so.
    \\v 10 God called the dry land “earth,” and the gathered waters he called “seas.” He saw that it was good.

    \\v 11 God said, “Let the earth sprout vegetation: plants yielding seed and fruit trees bearing fruit whose seed is in the fruit, each according to its own kind.” It was so.
    \\v 12 The earth produced vegetation, plants producing seed after their kind, and trees bearing fruit whose seed was in it, after their kind. God saw that it was good.
    \\v 13 This was evening and morning, the third day.

    \\p
    \\v 14 God said, “Let there be lights in the sky to divide the day from the night and let them be as signs, for seasons, for days and years.
    \\v 15 Let them be lights in the sky to give light upon the earth.” It was so.

    \\v 16 God made the two great lights, the greater light to rule the day, and the lesser light to rule the night. He made the stars also.
    \\v 17 God set them in the sky to give light upon the earth,
    \\v 18 to rule over the day and over the night, and to divide the light from the darkness. God saw that it was good.
    \\v 19 This was evening and morning, the fourth day.

    \\p
    \\v 20 God said, “Let the waters be filled with great numbers of living creatures, and let birds fly above the earth in the expanse of the sky.”
    \\v 21 God created the great sea creatures, as well as every living creature after its kind, creatures that move and which fill the waters everywhere, and every winged bird after its kind. God saw that it was good.

    \\v 22 God blessed them, saying, “Be fruitful and multiply, and fill the waters in the seas. Let birds multiply on the earth.”
    \\v 23 This was evening and morning, the fifth day.

    \\p
    \\v 24 God said, “Let the earth produce living creatures, each according to its own kind, livestock, creeping things, and beasts of the earth, each according to its own kind.” It was so.
    \\v 25 God made the beasts of the earth after their kind, the livestock after their kinds, and everything that creeps upon the ground after its kind. He saw that it was good.

    \\v 26 God said, “Let us make man in our image, after our likeness. Let them have dominion over the fish of the sea, over the birds of the sky, over the livestock, over all the earth, and over every creeping thing that creeps on the earth.” \\f + \\ft Some ancient manuscripts have \\fqa …Over the livestock, over all the animals of the earth, and over every creeping thing that creeps on the earth \\fqa* . \\f*
    \\v 27 God created man in his own image. In his own image he created him. Male and female he created them.

    \\v 28 God blessed them and said to them, “Be fruitful, and multiply. Fill the earth, and subdue it. Have dominion over the fish of the sea, over the birds of the sky, and over every living thing that moves upon the earth.”
    \\v 29 God said, “See, I have given you every plant yielding seed which is upon the surface of all the earth, and every tree with fruit which has seed in it. They will be food to you.

    \\v 30 To every beast of the earth, to every bird of the heavens, and to everything that creeps upon the earth, and to every creature that has the breath of life I have given every green plant for food.” It was so.
    \\v 31 God saw everything that he had made. Behold, it was very good. This was evening and morning, the sixth day.

    \\c 2
    \\p
    \\v 1 Then the heavens and the earth were finished, and all the living things that filled them.
    \\v 2 On the seventh day God came to the end of his work which he had done, and so he rested on the seventh day from all his work.
    \\v 3 God blessed the seventh day and sanctified it, because in it he rested from all his work which he had done in his creation.

    \\p
    \\v 4 These were the events concerning the heavens and the earth, when they were created, on the day that Yahweh God made the earth and the heavens.
    \\v 5 No bush of the field was yet in the earth, and no plant of the field had yet sprouted, for Yahweh God had not caused it to rain upon the earth, and there was no man to cultivate the ground.
    \\v 6 But a mist went up from the earth and watered the whole surface of the ground.`


If you log the sample itself to the console (:code:`console.log(sample)`), it is clear that the double slashes are not actually
part of the text.
