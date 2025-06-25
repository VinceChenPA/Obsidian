# Apache Iceberg

## Data Layer

### Data files

Apache Iceberg is file format agnostic and currently supports Apache Parquet, Apache ORC, and Apache Avro.

### Delete files

- delete files is only supported in the Iceberg v2 format
- only apply to MOR tables

#### COW (Copy on Write)

a copy of old file with changes reflected in the new file

#### MOR (Merge on Read)

a new file that only has the changes written, which engines reading the data then coalesce

#### positional delete files

#### equality delete files

## Metadata Layer

### Manifest files

- keep track of files in the data layer (i.e., datafiles and delete files) as well as additional details and statistics about each file, such as the minimum and maximum values for a datafile’s columns.
- in Iceberg, manifest files are in Avro format

### Manifest Lists

- A manifest list is a snapshot of an Iceberg table at a given point in time.
- contains a list of all the manifest files, including the location, the partitions it belongs to, and the upper and lower bounds for partition columns for the datafiles it tracks.
