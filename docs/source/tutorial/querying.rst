.. _querying:

#################
Querying the data
#################

Now that the data is loaded in Proskomma, it can be accessed using a GraphQL api.
We will look at a number of simple examples, before explaining the basic building 
blocks such as Sequence, Item, or Block that you might need to query Proskomma.

A query is a string that matches the GraphQL definitions provided by Proskomma out of the box.

Because Proskommma uses GraphQL, we will introduce some basic ideas that are needed to search for
data using GraphQL. Here are a few basic ideas that underlie GraphQL. They might be just enough to get you
through this tutorial. 

- For GraphQL searches every field needs to be explicitly mentioned.
- Rather than having a large series of API endpoints (as we could have in a REST api), there essentially is one api endpoint that allows the developer to access a wide range of data. Behind the scenes some of the work is done by different parts of the GraphQL implementation.
- Rather than performing multiple requests to get related data, a single request can contain multiple queries. Each one of these queries can be nested. For instance, the query :code:`'{ docSets { id } }'` gets the docSets and for each docSet gets its id. 
- GraphQL looks the way it does because it borrows an existing mechanism from Javascript - see `Nested object and array destructuring: <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#object_destructuring>`_. That is in essence what is happining with the query :code:`{ id }`.

GraphQL queries can get quite complex, which is why a good understanding of the underlying api implementation is important. 
What follows is a description of the query language that Proskomma uses and not its internals. 

Proskomma has a method called `gqlQuery` which allows you to run a GraphQL query. 

If you were to run a query without handling the fact that it yields a promise the result would be the following:

.. code:: javascript

    console.log(pk.gqlQuery('{ docSets { id } }'))
    Promise { <pending> }

Remove the :code:`console.log()` above. 

Rather than getting the actual result we are getting a javascript Promise back. To fix this, let's 
embed the query in an async function and get its result:

.. code:: javascript

    const queryPk = async function (query) {
    const result = await pk.gqlQuery(query);
        console.log(JSON.stringify(result, null, 2));  // we are converting the result to an indented string, in production code you'll want to work with an actual object
        }

Using the above function, we can start querying Proskomma, using standard
GraphQL queries 

Let's get the ids of the docSets

.. code:: javascript

    queryPk('{ docSets { id } }')

Because we only have a single docSet, the result is 

.. code:: javascript

    {"data":{"docSets":[{"id":"eng_ult"}]}}

Let's go through a series of queries that are intended to show how to get data out of Proskomma. We'll start with the simplest possible query.

.. code:: javascript

    queryPk('{ id }')

This query is only for debugging purposes. It returns a unique id for your Proskomma instance, 
for instance, in case you have multiple instances of Proskomma running. 

Space separate multiple fields if you want to retrieve multiple ones

.. code:: javascript

    queryPk('{ processor packageVersion }')

=====================
docSets and documents
=====================

Generally, a docSet would be a Bible, and a document a book in that Bible
But a docSet could be any collection of documents you choose.

To get the count of number of Documents that were loaded in Proskomma, use `nDocuments`
the `n` stands for 'the number of'

.. code:: javascript

    queryPk('{ nDocuments }')

You can get multiple resources at the same time, so here you can get the number of documents
and the number of docSets

.. code:: javascript

    queryPk('{ nDocuments nDocSets }')

The information in Proskomma is structured in a tree-like way: docSets contain documents, which
contain sequences, and each of these sequences contain blocks, and these blocks have a scope, and 
they can contain grafts. 

You can go down the tree either via documents or via docSets. These are the two entry points

.. code:: javascript

    queryPk('{ documents {id} }')
    queryPk('{ docSets {id} }')

You can go one level down as follows; to get the docSets id's, but also the id's of the documents it contains

.. code:: javascript

    queryPk('{ docSets { id documents { id } } }')

And again you can obtain more data points at the same time, for instance by adding nDocuments at the same level as documents

.. code:: javascript

    queryPk('{ docSets { id nDocuments documents { id } } }')

Curly braces take you one level down.

Let's get the header for our sample document. 

In this case we need to go three levels down: docSets -> documents -> headers

.. code:: javascript

    queryPk('{ docSets { id documents { id headers { key value } } } }')

key and value are not the traditional key-value pairs you might think of, for instance, as in 
a Python dictionary. They are keywords here. GraphQL needs you to specify every field, so if you do not
know which exact part of the header you want, you can use `key value` to get all of them.
so here key and value are the fields that you query for

Alternatively, you could get a specific header
header() here is a field that takes a compulsary argument
which is why it has the round brackets.

.. code:: javascript

    queryPk('{ docSets { id documents { id header(id:"toc") } } }')

As our query gets large, let's split it over multiple lines.

.. code:: javascript

    queryPk(`
    { docSets 
        { id 
        documents 
        { id 
            header(id:"toc") 
        } 
        } 
    }`)

You can also get multiple headers, in this case you need rename the field (give it an alias)
by adding a `alias:` before the name of the field. You are free to choose any name you like for your aliases.
For instance, the `title` and `shortName` aliases below can easily be changed.

.. code:: javascript

    queryPk(`
    { docSets 
        { id 
        nDocuments
        documents 
        { id 
            title: header(id:"toc")      
            shortName: header(id:"toc2") 
        } 
        } 
    }
    `)

you can also filter the docSets based on which book they contain (which is not helpful in our case, but
it demonstrates the use)

.. code:: javascript

    queryPk(`
    { docSets(withBook: "GEN")
        { id 
        nDocuments
        documents 
        { id 
            title: header(id:"toc")      
            shortName: header(id:"toc2") 
        } 
        } 
    }
    `)

If you were to have multiple documents in your docSet (unlike our simple example here)
you could retreive the information for that specific book/document.
Note how we are using `document` here and not `documents` in plural

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { id 
        title: header(id:"toc")      
        shortName: header(id:"toc2") 
        } 
    } 
    }
    `)

=========
Sequences
=========
    

The next level down from documents is sequences. 
A sequence is a flowing text. It is a text in its own right.

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        sequences { id type }
        } 
    } 
    }
    `)

The main sequence is the actual text of the Bible.
All the other sequences are based on main. 

Use WordLikes on the mainSequence to get its vocabulary.

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        mainSequence { id type wordLikes tags }
        } 
    } 
    }
    `)

Or check if the mainSequence has specific characters

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        mainSequence { id type tags hasChars(chars: "God")}
        } 
    } 
    }
    `)

There is only one mainSequence that contains the entire main text of a document.
Other sequences, such as footnotes, form distinct sequences. Each footnote is its own sequence.
In fact, each sequence can be as complex as needed, meaning it is 
it's a tree in its own right. That means that it can contains its own complexity. 

======
Blocks
======

The next level in the tree are blocks.
Blocks are paragraphs in the USFM sense of that word, meaning they do not
correspond to paragraphs in HTML or in a Word-processor. Almost everything is
a paragraph in USFM, for instance, even the elements in the header of the file
are considered paragraphs. Because almost everything is a paragraph in USFM, Proskomma
calls these blocks and not paragraphs.

We now have docSets -> documents -> sequences -> blocks

A sequence is a list of blocks with metadata attached to it.
To get all the sequences and for each sequence how many blocks it contains, run


.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        sequences { id type nBlocks }
        } 
    } 
    }
    `)

----------------
The mainSequence
----------------

For the mainSequence we can now get its blocks, and specifically the text of the paragraphs
This is, finally, the way to retrieve the actual Biblical text we imported at the start of the 
tutorial. 

The mainSequence contains everything, but for footnotes, for instance, these are just references 
to the actual footnotes

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        mainSequence { 
            id type blocks { 
            text 
            } 
        }
        } 
    } 
    }
    `)

You can get all blocks of all sequences

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        sequences { 
            id type blocks {  
            text 
            } 
        }
        } 
    } 
    }
    `)

You can also get a specific block in your result 

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        mainSequence { id type blocks(positions: 0) { text } }
        } 
    } 
    }
    `)

=================
Scopes and Grafts
=================
    

This is were things start to become slight more complex, but for a good reason.
Retrieving the text of the Bible is a relatively trivial task. In comparison with other
tools it might seem that the query above is quite elaborate just to retrieve the text. 
The data model of Proskomma, however, makes it possible to get all the footnotes, sidebars, and extra
information that is related to the main text. It also captures at any given point in the text
in what scope one is and it allows to retrieve specific attributes for each token. We will unpack
some of this functionality one step at a time.

Now lets add two extra fields that show the potential of having both the text and all its
related information in a single query interface.

For each block we will retrieve two extra things: 1) its block scope, and 2) its grafts. 

Its block scope is a way to retrieve where the block was found. In simple terms, scopes tell you where you are
It is shortened as `bs` for blockScope.
The last item in the following query gives us the following bs; the block scope tells us what type of paragraph we're in.
Alternative bs's could also be elements to indicate poetic language such as `\q1`.

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        mainSequence { 
            id type blocks { 
            text 
            bs { payload }
            } 
        }
        } 
    } 
    }
    `)

.. code:: javascript

    {
        "text": "These were the events concerning the heavens and the earth, when they were created, on the day that Yahweh God made the earth and the heavens.No bush of the field was yet in the earth, and no plant of the field had yet sprouted, for Yahweh God had not caused it to rain upon the earth, and there was no man to cultivate the ground.But a mist went up from the earth and watered the whole surface of the ground.",
        "bs": {
        "payload": "blockTag/p"
        },
        "bg": []
    }

The payload is something like "the main content". For tokens, it's a fragment of text. For scopes, it's the description of that scope, while for grafts it's the id of the destination sequence.

A graft is another sequence that is inserted at a specific point in the given block. Footnotes, for instance, 
are grafts. But certain headings could also be seen as grafts.

With both scopes and grafts, all the basic building blocks of the marked-up text are now available.

The following query gives us access to a lot of information.

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        mainSequence { 
            id type blocks { 
            text 
            bs { payload }
            bg { type subType payload }
            } 
        }
        } 
    } 
    }
    `)

The first block in the result of this query is the following:

.. code:: javascript

    {
        "text": "In the beginning, God created the heavens and the earth. \nThe earth was without form and empty. Darkness was upon the surface of the deep. The Spirit of God was moving above the surface of the waters.",
        "bs": {
        "payload": "blockTag/p"
        },
        "bg": [
        {
            "type": "graft",
            "subType": "title",
            "payload": "1zjRW_8fGzal"
        }
        ]
    },

We can learn multiple things from this result. Firstly, the result is not a single verse, but rather a full block paragraph. Secondly,
the scope of this block is that of a `blockTag/p` meaning it is a paragraph indeed. Thirdly, it contains a block graft, 
which in this case is a title. 

(It is also possible to structure the GraphQL output by chapter, by chapter/verse and according to particular markup such as the \ts milestone.)

There are two types of grafts in Proskomma: the block graft and the inline graft. The block graft is essentially a block that 
is inserted before the block it is attached to: for instance, the title is a block graft. It is a way of saying 'insert this content here' where
the content is a block in its own right. The inline graft is inserted at a given place *within* a block. One could think of the 
inline graft as a `span` in html. Footnotes are inline grafts, for instance.

We now have a lot of information and a hierarchy contains: docSets -> documents -> sequences -> blocks -> grafts and scopes

=====
Items
=====

There is one final thing we need to understand before ending this tutorial: a text is parsed into a smaller series of `items`. 
Almost everything is an item, including the words, punctuation, and whitespace (later these three are referred to as tokens), but 
also milestones, tags, and attributes. When you want to get access to these, there are two often used fields: tokens and items.

As a text is parsed into words, punctuation, or whitespace, these elements can be accessed using the tokens field. 

Let's use the tokens field to get the actual tokens of the first block of the main sequence using the positions field. 
The positions field takes an array as input, but you can also specificy you only want the first element.

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        mainSequence { 
            id type blocks(positions: 0) {  
            tokens { type subType payload}
            } 
        }
        } 
    } 
    }
    `)

.. code:: javascript


    {
        "data": {
        "docSets": [
            {
            "document": {
                "title": "The Book of Genesis",
                "mainSequence": {
                "id": "4L..9YzMFja4",
                "type": "main",
                "blocks": [
                    {
                    "tokens": [
                        {
                        "type": "token",
                        "subType": "wordLike",
                        "payload": "In"
                        },
                        {
                        "type": "token",
                        "subType": "lineSpace",
                        "payload": " "
                        },
                        {
                        "type": "token",
                        "subType": "wordLike",
                        "payload": "the"
                        },
                        {
                        "type": "token",
                        "subType": "lineSpace",
                        "payload": " "
                        },
                        {
                        "type": "token",
                        "subType": "wordLike",
                        "payload": "beginning"
                        },
                        {
                        "type": "token",
                        "subType": "punctuation",
                        "payload": ","
                        },
                        ... [ result truncated ]

We can see in the result above that the phrase 'In the beginning,' which starts our sample text,
is parsed into wordLike, lineSpace, and punctuation tokens. 

If you want to get not only the tokens, but also the grafts and their scopes, use `items` rather than
`tokens`. Items get you even more information.

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        sequences { 
            id type blocks(positions:0) {  
            items { type subType payload}
            } 
        }
        } 
    } 
    }
    `)

If tokens have attributes, these attributes are encoded as scopes. This might seem a bit counter-intuitive,
but it makes sense: a token can have multiple scopes, and hence multiple attributes. Our sample document
does not contain a lot of attributes (such as lemma or morphology in certain source texts), so this is left
for further exploration.
Note the `scopes` and `position` fields.

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        sequences { 
            id type blocks {  
            tokens { payload scopes position }
            } 
        }
        } 
    } 
    }
    `)

It is also possible to filter on an attribute:

.. code:: javascript

    queryPk(`
    { docSets
    { 
        document(bookCode:"GEN")
        { title: header(id:"toc")      
        sequences { 
            id type blocks {  
            tokens { payload scopes(startsWith: "verse/") position }
            } 
        }
        } 
    } 
    }
    `)
