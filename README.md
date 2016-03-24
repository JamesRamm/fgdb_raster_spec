File Geodatabase Raster Specification
=====================================
This is an ongoing attempt to decipher the storage format for rasters' in an ESRI File Geodatabase.

Test Data
==========

``empty.gdb`` - empty geodatabase, what gets created when creating an new geodatbase through arccatalog
``empty_raster.gdb`` - same as above but with an empty raster added. The raster has a cell size of 5m, EPSG:27700 and no compression.

Raster Files
============
The following files are present in ``empty_raster.gdb`` but not ``empty.gdb``

|Name|Empty size|
|:---:|:-------:|
|a0000000a.gdbtable| 280  |
|a0000000b.gdbtable |138  |
|a0000000c.gdbtable |225  |
|a0000000d.gdbtable |598 |
|a00000009.gdbtable |2195 |
|a0000000c.blk_key_index.atx |4118 |
|a0000000c.col_index.atx |4118 |
|a0000000c.row_index.atx |4118 |
|a0000000c.band_index.atx |4118 |
|a0000000a.gdbindexes| 70  |
|a0000000b.gdbindexes| 66 |
|a0000000c.gdbindexes| 308 |
|a0000000d.gdbindexes| 86   |
|a00000009.gdbindexes| 112 |
|a00000009.gdbtablx |5152 |
|a0000000a.gdbtablx |5152 |
|a0000000b.gdbtablx |32 |
|a0000000c.gdbtablx |32  |
|a0000000d.gdbtablx |5152 |
|a00000009.spx| 4118 |

The above files are therefore the set of tables for a single raster.

Adding new rasters to the same GDB causes another set of 5 tables (each table has a .gdbtable, .gdbindexe and .gdbtablx) to be created.
They appear to be numbered/lettered sequentially although it is not clear how it is decided whether a table ends with a number of letter.

The .gdbtable containing the actual pixel data shares the same name as the .blk_key_index.atx/.col_index.atx/.row_index.atx/.band_index.atx files, which
appear once per dataset.

Table Files
===========

From the above references, the `dump_gdb.py` tool created for the FGDB vector reverse engineering and by creating small rasters with tiny changes,
we can deduce structure/heirarchy of the .gdbtable files (these files contain the actual data).

a0000000a.gdbtable - Business Table/Raster Table
-------------------
Fields:

- `raster_id` Primary key of the raster table 
- `raster_flags`'bit map set according to characteristics of stored image'
- `description`? presuambly can store a string description
- `storage_def` Binary field, perhaps whether it is a managed raster, moasiac, 'raster as attribute'?

a0000000b.gdbtable - Raster Auxilliary Table
-------------------


Fields:

- `rasterband_id` 
    Foriegn key to the Raster Band tables' primary key.
    
- `type`
    'Bit map set according to the characteristics of data store in the object column'
    
- `object`
    ?? binary blob. Perhaps for statistics/colour maps


a0000000c.gdbtable - Raster Block Table
-------------------

Fields:

- `rasterband_id`
- `rrd_factor`
    'Reduced resolution dataset factor', determines position of the raster band block within the resolution pyramid. 0 For highest resolution
- `row_nbr`
    The block's row number
- `col_nbr`
    The blocks' column number
- `block_data`
    the pixel data for this block. Binary blob. Appears to have some form of header (?) before the actual data
- `block_key`

a0000000d.gdbtable - Raster Band Table
-------------------


Fields:

- `rasterband_id`
    The primary key of the raster band table that uniquely identifies each raster band
- `sequence_nbr`
    An optional sequential number that can be combined with the raster_id as a composite key as a second way to uniquely identify the raster band
- `raster_id`
    The foreign key reference to the raster tables primary key Uniquely identifies the raster band when combined with the sequence_nbr as a composite key
- `name`
    The name of the raster band
- `band_flags`
    A bit map set according to the characteristics of the raster band
- `band_width`
    The pixel width of the band
- `band_height`
    The pixel height of the band
- `band_types`
    A bitmap band compression data
- `block_width`
    The pixel width of the band's tiles
- `block_height`
    The pixel height of the band's tiles
- `block_origin_x`
    The left-most pixel
- `block_origin_y`
    The bottom-most pixel
- `eminx`
    The band's minimum X coordinate
- `eminy` 
    The band's minimum Y coordinate
- `emaxx`
    The band's maximum X coordinate
- `emaxy`
    The band's maximum Y coordinate
- `cdate`
    The creation date
- `mdate`
    The last modification date
- `srid`
    Some identifier for the SRID code. It is *not* EPSG


a00000009.gdbtable
-------------------
Raster Geometry?
Appears to store info on the geometry of the raster (bounding box, Area, 'Length'?)


References
===========
This work has largely been pieced together by the amazing work done here: https://github.com/rouault/dump_gdbtable
and various documentation snippets published by ESRI.

This page in particular gives an interesting snippet:
http://desktop.arcgis.com/en/arcmap/10.3/manage-data/raster-and-images/how-raster-data-is-stored-and-managed.htm

'The storage model of the file geodatabases is a hybrid of the enterprise geodatabase and the personal geodatabase'.
The Enterprise Geodatabase (ArcSDE) schema is fairly well documented here:
http://edndoc.esri.com/arcsde/9.2/concepts/rasters/dbschema/dbschema_diagram.htm

The personal geodatbaase is an MS Access database, so the schema can be inspected in MSAccess.

https://blogs.esri.com/esri/arcgis/2010/03/15/the-simplified-geodatabase-schema-in-arcgis-10/
http://edndoc.esri.com/arcsde/9.2/concepts/rasters/dbschema/dbschema_diagram.htm
http://help.arcgis.com/EN/ArcGISDesktop/10.0/Help/index.html#//009t0000002z000000
https://github.com/rouault/dump_gdbtable/wiki/FGDB-Spec
