.. _example:

Example
!!!!!!!

This section provides a complete, language-neutral example of
essential features of VRS. In this example, we will translate an
HGVS-formatted variant, ``NC_000019.10:g.44908822C>T``, into its VRS
format and assign a globally unique identifier.

Translate HGVS to VRS
@@@@@@@@@@@@@@@@@@@@@

The |varnomen| string ``NC_000019.10:g.44908822C>T`` represents a
single base substitution on the reference sequence `NC_000019.10
<https://www.ncbi.nlm.nih.gov/nuccore/NC_000019.10>`_ (human
chromosome 19, assembly GRCh38) at position 44908822 from the
reference nucleotide ``C`` to ``T``.

In VRS, a contiguous change is represented using an :ref:`allele`
object, which is composed of a :ref:`Location <location>` and of the
:ref:`State <state>` at that location. Location and State are
abstract concepts: VRS is designed to accommodate many kinds of
Locations based on sequence position, gene names, cytogenetic bands, or
other ways of describing locations. Similarly, State may refer to a
specific sequence change, a contiguous repeated sequence, or a
sequence derived from another source.

In this example, we will use a :ref:`SequenceLocation`, which is
composed of a :ref:`SequenceReference` and start and end coordinates.

In VRS, the :ref:`SequenceReference` object's *refgetAccession*
attribute MUST use a `GA4GH RefGet
<http://samtools.github.io/hts-specs/refget.html>`_ identifier.
Therefore, ``NC_000019.10`` MUST be written as the string
``SQ.IIB53T8CNeJJdUqzn9V_JnRtQadwWCbl``.

VRS uses :ref:`inter-residue-coordinates-design`. Inter-residue
coordinates *always* use intervals to refer to sequence spans. For
the purposes of this example, inter-residue coordinates *look* like the
more familiar 0-based, right-open numbering system. (Please read
about :ref:`inter-residue-coordinates-design` if you are interested in
the significant advantages of this design choice over other coordinate
systems.)

The :ref:`SequenceLocation` for the position ``44908822`` is:

.. code-block:: json

    {
      "type": "SequenceLocation",
      "sequenceReference": {
          "type": "SequenceReference",
          "refgetAccession": "SQ.IIB53T8CNeJJdUqzn9V_JnRtQadwWCbl"
      },
      "start": 44908821,
      "end": 44908822
    }

A :ref:`LiteralSequenceExpression` object consists simply of the replacement sequence, as follows:

.. code-block:: json

    {
      "sequence": "T",
      "type": "LiteralSequenceExpression"
    }

The :ref:`Allele` object's *location* and *state* attributes may
then be constructed from the above SequenceLocation and
LiteralSequenceExpressions respectively:

.. code-block:: json

    {
      "location": {
        "type": "SequenceLocation",
        "sequenceReference": {
            "type": "SequenceReference",
            "refgetAccession": "SQ.IIB53T8CNeJJdUqzn9V_JnRtQadwWCbl"
        },
        "start": 44908821,
        "end": 44908822
      },
      "state": {
        "sequence": "T",
        "type": "LiteralSequenceExpression"
      },
      "type": "Allele"
    }


This Allele is a fully-compliant VRS object that is parsable using the
VRS JSON Schema.

.. note:: VRS is verbose! The goal of VRS is to provide a extensible
          framework for representation of sequence variation in
          computers. VRS objects are readily parsable and have precise
          meaning, but are often larger than other representations and
          are typically less readable by humans. This tradeoff is
          intentional!



Generate a computed identifier
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

A key feature of VRS is an easily-implemented algorithm to
generate computed, digest-based identifiers for variation objects.
This algorithm permits organizations to generate the same identifier
for the same allele without prior coordination, which in turn
facilitates sharing, obviates centralized registration services, and
enables identifiers to be used in secure settings (such as diagnostic
labs).

The VRS computed identifier procedure requires that all nested
:term:`identifiable objects <identifiable object>` are expressed using
computed identifiers. Using GA4GH sequence identifiers collapses
differences between alleles due to trivial differences in reference
naming. The same variation reported on NC_000019.10, CM000681.2,
GRCh38:19, GRCh38.p13:19 would appear to be distinct variation; using
a digest identifier will ensure that variation is reported on a single
sequence identifier. Furthermore, using digest-based sequence
identifiers enables the use of custom reference sequences.

.. important:: VRS permits the use of conventional sequence accessions
               from RefSeq, Ensembl, or other sources by annotating the
               :ref:`SequenceReference` object's *id* attribute. When
               generating computed identifiers, the
               :ref:`SequenceReference` object's *refgetAccession*
               attribute MUST use a `GA4GH RefGet
               <http://samtools.github.io/hts-specs/refget.html>`_
               identifier.

The first step in constructing a computed identifier is to create a
binary digest serialization of the Allele. Details are provided in
:ref:`computed-identifiers`. For this example, the *binary* (ASCII
encoded) object looks like:

.. code-block:: text

   {"location":"wIlaGykfwHIpPY2Fcxtbx4TINbbODFVz","state":{"sequence":"T","type":"LiteralSequenceExpression"},"type":"Allele"}

.. important:: The GA4GH binary digest serialization process imposes
               constraints that guarantee that different
               implementations will generate the same binary "blob"
               for a given object. Do not confuse binary digest
               serialization with JSON serialization, which is used
               elsewhere with VRS schema.

The GA4GH digest for the above blob is computed using the first 192
bits (24 bytes) of the `SHA-512`_ digest, `base64url`_ encoded.
Conceptually, the function is ``base64url( sha512( blob )[:24] )``. In
this example, the value returned is
``0AePZIWZUNsUlQTamyLrjm2HWUw2opLt``.

A GA4GH Computed Identifier has the form::

  "ga4gh" ":" <type_prefix> "." <digest>

The ``type_prefix`` for a VRS Allele is ``VA``, which results in the
following computed identifier for our example::

  ga4gh:VA.0AePZIWZUNsUlQTamyLrjm2HWUw2opLt

Importantly, GA4GH computed identifiers may be used literally (without
escaping) in URIs.

Variation and Location objects contain an OPTIONAL *id* attribute
which implementations may use to store any CURIE-formatted identifier.
*If* an implementation returns a computed identifier with objects, the
object might look like the following:

.. code-block:: json

    {
      "id": "ga4gh:VA.0AePZIWZUNsUlQTamyLrjm2HWUw2opLt",
      "location": {
        "type": "SequenceLocation",
        "sequenceReference": {
            "type": "SequenceReference",
            "refgetAccession": "SQ.IIB53T8CNeJJdUqzn9V_JnRtQadwWCbl"
        },
        "start": 44908821,
        "end": 44908822
      },
      "state": {
        "type": "LiteralSequenceExpression",
        "sequence": "T"
      }
    }

This example provides a full VRS-compliant Allele with a computed identifier.

.. note:: The *id* attribute is optional. If it is used, the value
          MUST be a string, but it does NOT need to be a GA4GH Computed
          Identifier. Applications MAY choose to implement their own
          identifier scheme for private or public use. For example,
          the above *id* could be a serial number assigned by an
          application, such as ``acmecorp:v0000123``.


What's Next?
@@@@@@@@@@@@

This example has shown a full example for a relatively simple case.
VRS provides a framework that will enable much more complex variation.
Please see :ref:`future-plans` for a discussion of variation classes
that are intended in the near future.

The :ref:`implementations` section lists libraries and packages that
implement VRS.

VRS objects are `value objects
<https://en.wikipedia.org/wiki/Value_object>`__. An important
consequence of this design choice is that data should be associated
*with* VRS objects via their identifiers rather than embedded *within*
those objects. The appendix contains an example of :ref:`associating
annotations with variation <associating_annotations>`.
