.. _data:

################
Loading the data
################

We now instantiate Proskomma into a `pk` variable. The abbreviation `pk` is often used as shorthand for Proskomma, so we'll follow that practice.

.. code:: javascript

    const pk = new Proskomma();

We can now have a look at that instance, even before it contains any data:

.. code:: javascript

    console.log('Here is our instance of Proskomma, before importing any document')
    console.log(pk)

The output should look similar to what follows:

.. code:: javascript

    Here is our instance of Proskomma, before importing any document
    Proskomma {
        processorId: 'QKe5yV6qHZe6',
        documents: {},
        docSetsBySelector: {},
        docSets: {},
        filters: {},
        customTags: {
        heading: [],
        paragraph: [],
        char: [],
        word: [],
        intro: [],
        introHeading: []
        },
        emptyBlocks: [],
        selectors: [
        { name: 'lang', type: 'string', regex: '[a-z]{3}' },
        { name: 'abbr', type: 'string' }
        ],
        mutex: Mutex {
        _semaphore: Semaphore { _maxConcurrency: 1, _queue: [], _value: 1 }
        }
    }

Don't worry about the details of the internals. In general, you are going to use the GraphQL interface described below. However, note that you have an instance of a class, and that the top-level structure looks a lot like the corresponding GraphQL structure. (This rapidly ceases to be the case for finer-grained content.)

`pk` has a processerId and two default selectors (being `lang` for language and `abbr` for the abbreviation).
The selectors specify the docSets that are part of Proskomma, and not the actual documents.

Often a docSets is a version of the Bible, and the documents are the books of said Bible.

This instance of Proskomma (`pk`) does not contain any document set so far, so let
us read the above sample and store it as a document.

Proskomma has an importDocument function that can do this for us:

.. code:: javascript

    const pkDoc = pk.importDocument(
            selectors={lang: 'eng',  abbr: 'ult'},  // we need to give it some selectors so it can be retrieved
            contentType='usfm',                     // we need to provide it a contentType, this could also be `usx`, for instance
            content=sample,                         // we feed it the actual content, in this the `sample` defined above
        );

Again, in practice, most of the time we would be reading the content from a file on the filesystem or the content of an http request, 
but for this tutorial we'll simply work with a string as input.

The `importDocument` has additional arguments we do not use right now (options, customTags, emptyBlocks, tags)
The options argument in particular is useful for filtering out markup as part of the import process (eg to remove alignment data when it is not needed).

Now we have imported a document in Proskomma, let's look at `pk` again:

.. code:: javascript

    console.log('Here is pk, now after importing a document')
    console.log(pk);

.. code:: javascript

    Here is pk, now after importing a document
    Proskomma {
        processorId: 'FedDnnXSIWam',
        documents: {
        TGxAdND1GqiA: Document {
            processor: [Circular *1],
            docSetId: 'eng_ult',
            baseSequenceTypes: [Object],
            id: 'TGxAdND1GqiA',
            filterOptions: {},
            customTags: [Object],
            emptyBlocks: [],
            tags: Set(0) {},
            headers: [Object],
            mainId: 'evjC20YiIMaS',
            sequences: [Object]
        }
        },
        docSetsBySelector: { eng: { ult: [DocSet] } },
        docSets: {
        eng_ult: DocSet {
            processor: [Circular *1],
            preEnums: {},
            enumIndexes: [Object],
            docIds: [Array],
            selectors: [Object],
            id: 'eng_ult',
            tags: Set(0) {},
            enums: [Object]
        }
        },
        filters: {},
        customTags: {
        heading: [],
        paragraph: [],
        char: [],
        word: [],
        intro: [],
        introHeading: []
        },
        emptyBlocks: [],
        selectors: [
        { name: 'lang', type: 'string', regex: '[a-z]{3}' },
        { name: 'abbr', type: 'string' }
        ],
        mutex: Mutex {
        _semaphore: Semaphore { _maxConcurrency: 1, _queue: [], _value: 1 }
        }
    }

We now have a bit more information.
Most importantly, we can see that Proskomma now contains a document
and a docSet. The docSet is identified using the selectors we provided
as we were reading the document as `eng_ult` 

The pk object also has a number of methods that could be helpful when you
need to access, for instance, the number of documents. The main way of interacting 
with Proskomma, however, is not so much by handling the pk object itself, but rather
by using the query interface that we will introduce later in this tutorial.

docSets are combinations of documents. You can have multiple docSets in a Proskomma instance.

.. code:: javascript

    console.log('Here is more information on the document we read:')
    console.log(pkDoc);

Here is the result: 

.. code:: javascript

    Document {
        processor: Proskomma {
        processorId: '3jYqOOByHJXM',
        documents: { 'Y7uSbLv.IUX3': [Circular *1] },
        docSetsBySelector: { eng: [Object] },
        docSets: { eng_ult: [DocSet] },
        filters: {},
        customTags: {
            heading: [],
            paragraph: [],
            char: [],
            word: [],
            intro: [],
            introHeading: []
        },
        emptyBlocks: [],
        selectors: [ [Object], [Object] ],
        mutex: Mutex { _semaphore: [Semaphore] }
        },
        docSetId: 'eng_ult',
        baseSequenceTypes: {
        main: '1',
        introduction: '*',
        introTitle: '?',
        introEndTitle: '?',
        title: '?',
        endTitle: '?',
        heading: '*',
        header: '*',
        remark: '*',
        sidebar: '*'
        },
        id: 'Y7uSbLv.IUX3',
        filterOptions: {},
        customTags: {
        heading: [],
        paragraph: [],
        char: [],
        word: [],
        intro: [],
        introHeading: []
        },
        emptyBlocks: [],
        tags: Set(0) {},
        headers: {
        id: 'GEN EN_ULT en_English_ltr unfoldingWord Literal Text Thu Jul 25 2019 09:33:56 GMT-0400 (EDT) tc',
        bookCode: 'GEN',
        usfm: '3.0',
        ide: 'UTF-8',
        h: 'Genesis',
        toc: 'The Book of Genesis',
        toc2: 'Genesis',
        toc3: 'Gen'
        },
        mainId: '5YVuJg.FI6en',
        sequences: {
        '5YVuJg.FI6en': {
            id: '5YVuJg.FI6en',
            type: 'main',
            tags: Set(0) {},
            isBaseType: true,
            blocks: [Array],
            verseMapping: {},
            chapterVerses: [Object],
            tokensPresent: [Object],
            chapters: [Object]
        },
        S9l6VpnzHWyB: {
            id: 'S9l6VpnzHWyB',
            type: 'title',
            tags: Set(0) {},
            isBaseType: true,
            blocks: [Array]
        },
        OkULPDsdIKPQ: {
            id: 'OkULPDsdIKPQ',
            type: 'heading',
            tags: Set(0) {},
            isBaseType: true,
            blocks: [Array]
        },
        ...
        '1dNl7oDgGiuV': {
            id: '1dNl7oDgGiuV',
            type: 'footnote',
            tags: Set(0) {},
            isBaseType: false,
            blocks: [Array]
        },
        mnIjb5TrF85g: {
            id: 'mnIjb5TrF85g',
            type: 'noteCaller',
            tags: Set(0) {},
            isBaseType: false,
            blocks: [Array]
        }
        }
    }

We can see the docSets that Proskomma now contains, which is only one:
:code:`docSets: { eng_ult: [DocSet] },`

One of the key ideas behind Proskomma is the 'Sequence'.
In our document we can already see that there are a number of `baseSequenceTypes`
the most important one of which is automatically added: **main**.

Proskomma has also parsed the header of our USFM file.

.. code:: javascript

        headers: {
        id: 'GEN EN_ULT en_English_ltr unfoldingWord Literal Text Thu Jul 25 2019 09:33:56 GMT-0400 (EDT) tc',
        bookCode: 'GEN',
        usfm: '3.0',
        ide: 'UTF-8',
        h: 'Genesis',
        toc: 'The Book of Genesis',
        toc2: 'Genesis',
        toc3: 'Gen'
        }

The title of a document is a special case, because it is parsed separately into a title sequence.

Finally, we can see a list of sequences. We have removed the list of sequences above,
but the basic idea is quickly clear: besides the main sequence, we also have 
a title sequence, a heading sequence, and a footnote sequence, for instance.
This corresponds to our sample document which indeed contains a footnote in 
Genesis 1:26.
