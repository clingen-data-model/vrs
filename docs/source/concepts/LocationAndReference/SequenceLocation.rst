.. _SequenceLocation:

Sequence Location
!!!!!!!!!!!!!!!!!

The sequence location class is a fundamental concept in VRS. Sequence locations are used to describe every form of :ref:`Variation`,
and they have stand-alone utility for describing sequence locations in other (non-variation) contexts.
This class is used to represent a location on a specified :ref:`SequenceReference`. The sequence reference is typically a
chromosome, transcript, or protein sequence.

Definition and Information Model
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@

.. include::  ../../def/vrs/SequenceLocation.rst

Implementation Guidance
@@@@@@@@@@@@@@@@@@@@@@@

Start, End, and Ranges
######################

At least one of the *start* and *end* properties MUST be specified in any ``SequenceLocation`` instance.
When only one of these properties is specified, this represents an open interval beginning at the specified
coordinate and extending left (when *start* is ``null``) or right (when *end* is ``null``).

When there is ambiguity at a coordinate (e.g., when using a ``SequenceLocation`` to describe the confidence boundary
of a copy number segment), this is specified using the :ref:`Range` class for that coordinate.

.. admonition:: New in v2

    In VRS v1, the ``SequenceLocation`` class had an *interval* property which contained *start* and *end*
    attributes. This intermediate object layer has been removed in v2.0, making *start* and *end*
    top-level properties of the ``SequenceLocation``.

The "Ref" Allele
################

In some variant representation formats (e.g. HGVS, VCF) sequence variants are described by both their "reference"
(ref) and "alternate" (alt) alleles. When representing an Allele with VRS v2, it is also possible to describe the
ref sequence (derived from the :ref:SequenceReference at the location) using the *sequence* property.

.. admonition:: New in v2

   In VRS v1, sequence derived from the reference was not transmitted. This feature was added in VRS v2 due to
   this common practice in other variant representation formats.

Linear and Circular Sequence Coordinates
########################################

When representing a linear sequence, it is expected that for a :ref:`Sequence` of length *n*, ``0 ≤ start ≤ end ≤ n``

For a circular sequence, ``0 ≤ end ≤ start ≤ n`` is also allowed. In cases where ``end < start``, this represents
a location that spans the circular sequence origin coordinate.

.. admonition:: New in v2

    The v2 ``SequenceLocation`` now also supports circular sequences. The optional *circular* property of the
    :ref:`SequenceReference` class may be set to ``True`` or ``False`` to explicitly indicate if a reference is
    circular, and therefore if ``0 ≤ end ≤ start ≤ n`` is also allowed.

Implied Sequence Coordinates
############################

The ``Sequence Location`` class refers to coordinates on a :ref:`SequenceReference`; if that sequence
represents a coding transcript, then the coordinates refer to the coding transcript, and not a
chromosome sequence to which it aligns. VRS intentionally does not allow for *start* or *end* values
that use an offset system to represent sequence not found on the :ref:`SequenceReference`.

.. TODO:: Describe and add a ref to an intronic variant profile
