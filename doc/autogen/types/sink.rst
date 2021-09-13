.. rubric:: Methods

.. spicy:method:: sink::close sink close False void ()

    Closes a sink by disconnecting all parsing units. Afterwards the
    sink's state is as if it had just been created (so new units can be
    connected). Note that a sink is automatically closed when the unit it
    is part of is done parsing. Also note that a previously connected
    parsing unit can *not* be reconnected; trying to do so will still
    throw a ``UnitAlreadyConnected`` exception.

.. spicy:method:: sink::connect sink connect False void (u: strong_ref<unit>)

    Connects a parsing unit to a sink. All subsequent write operations to
    the sink will pass their data on to this parsing unit. Each unit can
    only be connected to a single sink. If the unit is already connected,
    a ``UnitAlreadyConnected`` exception is thrown. However, a sink can
    have more than one unit connected to it.

.. spicy:method:: sink::connect_filter sink connect_filter False void (filter: strong_ref<unit>)

    Connects a filter unit to the sink that will transform its input
    transparently before forwarding it for parsing to other connected
    units.

    Multiple filters can be added to a sink, in which case they will be
    chained into a pipeline and the data will be passed through them in
    the order they have been added. The parsing will then be carried out
    on the output of the last filter in the chain.

    Filters must be added before the first data chunk is written into the
    sink. If data has already been written when a filter is added, an
    error is triggered.

.. spicy:method:: sink::connect_mime_type sink connect_mime_type False void (inout mt: bytes, scope: string = hilti::linker_scope())

    Connects parsing units to a sink for all parsers that support a given
    MIME type. In order for parsers to be included they should either be
    ``public``, or have a linker scope matching the passed ``scope``; if
    no ``scope`` is given all ``public`` parsers and all parsers compiled
    together are considered. All subsequent write operations to the sink
    will pass their data on to these parsing units. The MIME type may have
    wildcards for type or subtype, and the method will then connect units
    for all matching parsers.

.. spicy:method:: sink::connect_mime_type sink connect_mime_type False void (mt: string, scope: string = hilti::linker_scope())

    Connects parsing units to a sink for all parsers that support a given
    MIME type. In order for parsers to be included they should either be
    ``public``, or have a linker scope matching the passed ``scope``; if
    no ``scope`` is given all ``public`` parsers and all parsers compiled
    together are considered. All subsequent write operations to the sink
    will pass their data on to these parsing units. The MIME type may have
    wildcards for type or subtype, and the method will then connect units
    for all matching parsers.

.. spicy:method:: sink::gap sink gap False void (seq: uint<64>, len: uint<64>)

    Reports a gap in the input stream. *seq* is the sequence number of the
    first byte missing, *len* is the length of the gap.

.. spicy:method:: sink::sequence_number sink sequence_number False uint<64> ()

    Returns the current sequence number of the sink's input stream, which
    is one beyond the index of the last byte that has been put in order
    and delivered so far.

.. spicy:method:: sink::set_auto_trim sink set_auto_trim False void (enable: bool)

    Enables or disables auto-trimming. If enabled (which is the default)
    sink input data is trimmed automatically once in-order and processed.
    See ``trim()`` for more information about trimming.

.. spicy:method:: sink::set_initial_sequence_number sink set_initial_sequence_number False void (seq: uint<64>)

    Sets the sink's initial sequence number. All sequence numbers given to
    other methods are then assumed to be absolute numbers beyond that
    initial number. If the initial number is not set, the sink implicitly
    uses zero instead.

.. spicy:method:: sink::set_policy sink set_policy False void (policy: enum)

    Sets a sink's reassembly policy for ambiguous input. As long as data
    hasn't been trimmed, a sink will detect overlapping chunks. This
    policy decides how to handle ambiguous overlaps. The default (and
    currently only) policy is ``ReassemblerPolicy::First``, which resolves
    ambiguities by taking the data from the chunk that came first.

.. spicy:method:: sink::skip sink skip False void (seq: uint<64>)

    Skips ahead in the input stream. *seq* is the sequence number where to
    continue parsing. If there's still data buffered before that position
    it will be ignored; if auto-skip is also active, it will be
    immediately deleted as well. If new data is passed in later that comes
    before *seq*, that will likewise be ignored. If the input stream is
    currently stuck inside a gap, and *seq* lies beyond that gap, the
    stream will resume processing at *seq*.

.. spicy:method:: sink::trim sink trim False void (seq: uint<64>)

    Deletes all data that's still buffered internally up to *seq*. If
    processing the input stream hasn't reached *seq* yet, parsing will
    also skip ahead to *seq*.

    Trimming the input stream releases the memory, but that means that the
    sink won't be able to detect any further data mismatches.

    Note that by default, auto-trimming is enabled, which means all data
    is trimmed automatically once in-order and processed.

.. spicy:method:: sink::write sink write False void (inout data: bytes, [ seq: uint<64> ], [ len: uint<64> ])

    Passes data on to all connected parsing units. Multiple *write* calls
    act like passing input in incrementally: The units will parse the
    pieces as if they were a single stream of data. If no sequence number
    *seq* is provided, the data is assumed to represent a chunk to be
    appended to the current end of the input stream. If a sequence number
    is provided, out-of-order data will be buffered and reassembled before
    being passed on. If *len* is provided, the data is assumed to
    represent that many bytes inside the sequence space; if not provided,
    *len* defaults to the length of *data*.

    If no units are connected, the call does not have any effect. If
    multiple units are connected and one parsing unit throws an exception,
    parsing of subsequent units does not proceed. Note that the order in
    which the data is parsed to each unit is undefined.

    .. todo:: The error semantics for multiple units aren't great.

.. rubric:: Operators

.. spicy:operator:: sink::Size uint<64> op:| t:sink op:|

    Returns the number of bytes written into the sink so far. If the sink
    has filters attached, this returns the value after filtering.

.. spicy:operator:: sink::Size uint<64> op:| t:strong_ref<sink> op:|

    Returns the number of bytes written into the referenced sink so far.
    If the sink has filters attached, this returns the value after
    filtering.

