=============
API reference
=============

File operations
---------------

.. function:: FITS(filename::String, mode::String="r")

   Open or create a FITS file. ``mode`` can be one of ``"r"``
   (read-only), ``"r+"`` (read-write) or ``"w"`` (write). In "write"
   mode, any existing file of the same name is overwritten.

   A ``FITS`` object is a collection of "Header-Data Units" (HDUs) and
   supports the following operations:

   - ``length(f::FITS)`` The number of HDUs in ``f``.
   - ``endof(f::FITS)`` Same as length.
   - ``f[i]`` Return the ``i``-th HDU.
   - ``f[name]`` or ``f[name, ver]`` Return the HDU containing the
     given the given HDUNAME (EXTNAME) keyword (an ASCIIString), and
     optionally the given HDUVER (EXTVER) number (an Integer).

.. function:: close(f::FITS)

   Close the file. Subsequent attempts to operate on ``f`` will result
   in an error. ``FITS`` objects are also automatically closed when
   they are garbage collected.

Header operations
-----------------

.. function:: read_header(hdu)

   Read the entire header from the given HDU and return a
   ``FITSHeader`` object. The value of each header record is parsed as
   ``Int``, ``Float64``, ``ASCIIString``, ``Bool`` or ``nothing``
   according to the FITS standard.  (If the value cannot be parsed
   according to the FITS standard, the value is stored as the raw unparsed
   ``ASCIIString``.)

.. function:: FITSHeader(keys, values, comments)

   Create a ``FITSHeader`` from arrays of keywords, values and comments.
   This type partially implements the Associative interface:

   * ``length(hdr)`` Number of records.
   * ``haskey(hdr)`` Header keyword exists.
   * ``keys(hdr)`` Array of keywords (not a copy).
   * ``values(hdr)`` Array of values (not a copy).
   * ``hdr[key]`` Get value based on keyword or index.
   * ``hdr[key] = value`` Set value based on keyword or index.

   Additionally, there are functions to get and set comments:

   * ``get_comment(hdr, key)`` Get the comment based on keyword or index.
   * ``set_comment!(hdr, key, comment)`` Set the comment baed on keyword
     or index.

.. function:: read_key(hdu, key)

   Read just the specified key and return a tuple of ``(value,
   comment)``.  The key can be either the index of the header record
   (Integer) or the header keyword (ASCIIString).


Image operations
----------------

.. function:: write(f::FITS, data::Array; header=nothing)

   Add a new ImageHDU to the file. The following array element types
   are supported: ``UInt8``, ``Int8``, ``UInt16``, ``Int16``,
   ``UInt32``, ``Int32``, ``Int64``, ``Float32``, ``Float64``. If a
   ``FITSHeader`` object is passed as the ``header`` keyword argument,
   the header will be added to the new HDU.

.. function:: read(hdu::ImageHDU)

   Read the entire image from disk.

.. function:: read(hdu::ImageHDU, range...)

   Read a subsection of the image from disk. E.g., ``read(hdu, 1:20, 1:2:20)``.

.. function:: ndims(hdu::ImageHDU)

   Get number of image dimensions, without reading the image into memory.

.. function:: size(hdu::ImageHDU)

   Get image dimensions, without reading the image into memory.

.. function:: size(hdu::ImageHDU, i::Integer)

   Get ``i``-th dimension.

.. function:: length(hdu::ImageHDU)

   Get total number of pixels in image (product of ``size(hdu)``).

.. function:: copy_section(hdu::ImageHDU, dest::FITS, r::Range...)

   Copy a rectangular section of an image and write it to a new FITS
   primary image or image extension. The new image HDU is appended to
   the end of the destination file; all the keywords in the input image
   will be copied to the output image. The common WCS keywords will be
   updated if necessary to correspond to the coordinates of the
   section. Examples:

   Copy the lower-left 200 x 200 pixel section of the image in ``hdu``
   to an open file, ``f``::
 
       copy_section(hdu, f, 1:200, 1:200)

   Same as above but only copy odd columns in y::

       copy_section(hdu, f, 1:200, 1:2:200)


Table Operations
----------------

.. function:: write(f::FITS, data::Dict; hdutype=TableHDU, extname=nothing, header=nothing, units=nothing)

   Create a new table extension and write data to it. If the FITS file
   is currently empty then a dummy primary array will be created
   before appending the table extension to it. ``data`` should be a
   dictionary with ASCIIString keys (giving the column names) and
   Array values (giving data to write to each column). The following
   types are supported in binary tables: ``Uint8``, ``Int8``,
   ``Uint16``, ``Int16``, ``Uint32``, ``Int32``, ``Int64``,
   ``Float32``, ``Float64``, ``Complex64``, ``Complex128``,
   ``ASCIIString``, ``Bool``

   Optional inputs:
   
   - ``hdutype``: Type of table extension to create. Can be either
     ``TableHDU`` (binary table) or ``ASCIITableHDU`` (ASCII table).
   - ``extname``: Name of extension.
   - ``header``: FITSHeader instance to write to new extension.
   - ``units``: Dictionary mapping column name to units (as a string).

.. function:: write(f::FITS, colnames, coldata; hdutype=TableHDU, extname=nothing, header=nothing, units=nothing)

   Same as ``write(f::FITS, data::Dict; ...)`` but providing column
   names and column data as a separate arrays. Column names must be
   ``Array{ASCIIString}`` and column data must be an array of
   arrays. Their lengths should match. This is useful for specifying
   the order of the columns.

.. function:: read(hdu, colname)

   Read a column as an array from the given table HDU.

   The column name may contain wild card characters (``*``, ``?``, or
   ``#``). The ``*`` wild card character matches any sequence of
   characters (including zero characters) and the ``?`` character
   matches any single character. The ``#`` wildcard will match any
   consecutive string of decimal digits (0-9). The string must match a
   unique column.

