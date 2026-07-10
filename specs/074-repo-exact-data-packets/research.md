# Research Decisions

## R1: Preserve Data wire, not reconstructed content

NDN Data signatures cover packet fields. Rebuilding a packet changes its wire
encoding and may change its name, metadata, FinalBlockId, or signature. The Repo
therefore stores and returns the original complete wire packet.

## R2: Separate logical object identity from Data packet identity

An application object can have a friendly logical name, but its transferred NDN
representation is a sequence of exact versioned Data names. A manifest relates
the two; it does not replace those names.

## R3: Exact name is the immutable deduplication key

Using `(objectName, segmentNo)` duplicates packets and permits conflicting
wires under one Data name. `dataName` as the primary key makes conflicts
explicit and permits safe sharing among manifests.

## R4: SegmentFetcher remains a consumer helper

SegmentFetcher can retrieve and assemble a versioned packet sequence. It does
not require or justify Repo-owned segment aliases. A manifest should expose the
original versioned name or exact packet names to the consumer.

## R5: Retain opaque-object compatibility separately

Some tests and applications store arbitrary bytes through STORE/FETCH. That
surface may remain, but its internal chunk names are not presented as canonical
NDN Data storage and new packet APIs do not use it.
