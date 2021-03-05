.. _under-hood-succinct_format:

###############
Succinct Format
###############

Low-level building blocks
=========================

The aim of the succinct format is to provide an in-memory data representation that can be read and modified without unpacking, and which does not require substantially more memory than the serialized representation. The succinct approach adopted here is based on two low-level representations: nBytes and counted strings.

nBytes
------

.. image:: ./nByte.svg
   :alt: nByte structure

An nByte provides variable-length storage of integers. 7 bits per byte are used for the value. The eight bit is a flag that shows whether the current byte is the last byte. Thus n bytes can store 128^n values, ie

- a 1-byte nByte can store 0-127
- a 2-byte nByte can store 0-16383
- a 3-byte nByte can store values up to about 2 million
- a 4-byte nByte can store values up to about 268 million

In practice, almost all Scripture chapter/verse numbers require only one byte, and 3 bytes is ample for almost any imaginable Scripture-related index. By contrast, a Javascript integer requires 8 bytes.

Counted Strings
---------------

.. image:: ./counted.svg
   :alt: Counted string structure

Counted strings are one of the standard ways to store textual data. For this application they have the advantage that it is easy to skip to the next string using the offset. That offset is one byte, limiting the maximum string length to 255 characters. Proskomma lexers split longer strings into several pieces.

Byte Arrays
===========

Byte arrays are based on uint8 typed arrays and are used to store Proskomma's higher-level succinct data structures such as items. Javascript typed arrays are fast and compact compared to the Array type. (They also offer less functionality since they are close to a C array, ie a block of memory accessible by index.)

Items
=====

.. image:: ./item.svg
   :alt: Item structure

Each item has a standard header which provides type information, as well as the length of the item record. It is therefore possible to filter and search blocks of items without examining every byte. The four types are

- token
- startScope
- endScope
- graft

The maximum record length is 63 bytes, two of which are used for the header. In practice this is quite generous as the payload almost always consists entirely of nBytes. (There is enough space for 15 4-byte nBytes or 20 3-byte nBytes.)

The nBytes in the payload generally point to counted strings. For tokens, that would be the actual textual content, which is stored once per docSet. For scopes, each nByte points to one part of the scope "path".

Blocks
======

Each block contains byte arrays representing:

- block scope (like a USFM paragraph style, ie "what kind of block is this?")
- block grafts (eg headings, introductions etc that could be inserted between blocks)
- content (items representing the content of the block)
- open scopes (which scopes were open at the start of the block)
- included scopes (which scopes are opened or closed within the block)

The last two byte arrays are intended to speed up search, and to enable a block to be rendered in context without parsing all preceding blocks.

Sequences
=========

Each sequence contains zero or more blocks. In future revisions it will also contain one or more set of block indexes for specific milestone-like markup. The first example of this will be chapter/verse ranges within the main sequence of a scripture document.
