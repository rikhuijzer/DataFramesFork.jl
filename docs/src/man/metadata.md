# Metadata

## Design of metadata support

DataFrames.jl allows you to store and retrieve metadata on table and column
level. This is supported using the functions defined by DataAPI.jl interface:
`hasmetadata`, `hascolmetadata`, `metadata` and `colmetadata`.
These functions work with `DataFrame`, `SubDataFrame`, `DataFrameRow`,
`GroupedDataFrame`, `DataFrameRows`, and `DataFrameColumns` objects.

Assume that we work with a data frame `df` that has a column `:col`.

### Contract for `hasmetadata`

In DataFrames.jl context `hasmetadata(df)` can return either `true` or `false`.
If `false` is returned this means that data frame `df` does not have any table
level metadata defined. If `true` is returned it means that at table level some
metadata is defined for `df`.

Although `hasmetadata` is guaranteed to return `Bool` value in DataFrames.jl
it is recommended to check its return value against `true` and `false` explicitly
using the `===` operator. The reason is that, in generic code, `hasmetadata`
is also allowed to return `nothing` if the queried object does not support
attaching metadata.

### Contract for `hascolmetadata`

In DataFrames.jl context `hascolmetadata(df, :col)` (alternatively string or
column number can be used) can return either `true` or `false`.
If `false` is returned this means that column `:col` of data frame `df` does not
have any column level metadata defined. If `true` is returned it means that for
`:col` column some metadata is defined.

Although `hascolmetadata` is guaranteed to return `Bool` value in DataFrames.jl
it is recommended to check its return value against `true` and `false` explicitly
using the `===` operator. The reason is that, in generic code, `hascolmetadata`
is also allowed to return `nothing` if the queried object does not support
attaching metadata.

### Contract for `metadata`

In DataFrames.jl `metadata(df)` always returns `Dict{String, Any}` storing
key-value mappings of table level metadata. To add or update metadata mutate
the returned dictionary.

### Contract for `colmetadata`

In DataFrames.jl `colmetadata(df, :col)` (alternatively string or
column number can be used) always returns `Dict{String, Any}` storing
key-value mappings of column level metadata for column `:col`.
To add or update metadata mutate the returned dictionary.

### General design principles for use of metadata

It is recommended to use strings as values of the metadata. The reason
is that some storage formats, like for example Apache Arrow, only support
storing string data as metadata values.

The `hasmetadata`, `hascolmetadata`, `metadata` and `colmetadata` functions
called on objects defined in DataFrames.jl are not thread safe and should not be
used in multi-threaded code. In particular, as an implementation detail, the
first time `metadata` is called on a data frame object, it will mutate it (this
might change in the future versions of DataFrames.jl).

In generic code it is recommended to run `hasmetadata` check before calling
`metadata` function. The reason is, that if some object does not support
metadata the call to `metadata` function throws an error. A similar rule
applies to calling `hascolmetadata` before calling `colmetadata`.

## Examples

Here is a simple example how you can work with metadata in DataFrames.jl:

```jldoctest dataframe
julia> df = DataFame(name="Jan Krzysztof Duda", date=, rating=)

julia> hasmetadata(df)

julia> df_meta = metadata(df)

julia> df_meta["name"] = "ELO ratings of chess players"

julia> hasmetadata(df)

julia> metadata(df)

julia> empty!(df_meta)

julia> hasmetadata(df)

julia> metadata(df)

julia> hasmetadata(df, :rating)

julia> rating_meta = metadata(df, :rating)

julia> rating_meta["name"] = "ELO rating in classical time"

julia> hasmetadata(df, :rating)

julia> metadata(df, :rating)

julia> empty!(rating_meta)

julia> hasmetadata(df, :rating)

julia> metadata(df, :rating)
```

## Propagation of metadata

An important design feature of metatada is how it is handled when you transform
data frames.

!!! note

    The provided rules might change in the future. Any change to metadata
    propagation rules will not be considered to be a breaking change
    in DataFrames.jl and can be done in any minor release of DataFrames.jl.
    Such changes might be made based on users' feedback about what metadata
    propagation rules are most convenient in practice.

Two general design rules for propagation of table and column level metadata
are as follows:
* if some object (data frame or column) is mutated in-place its metadata
  is not changed.
* if some object is moved as a whole (either by coping or by aliasing)
  then its metadata is propagated.
* in all other cases if a new object is created from an existing object and
  this transformation is not a copy or aliasing then metadata is dropped.

Here are the rules that currently are followed by DataFrames.jl.

### Operations that preserve table-level metadata

* `dropmissing!`
* `filter!`
* `insertcols!`
* `invpermute!`
* `permute!`
* `reverse!`
* `setindex!`
* `shuffle!`
* `unique!`

### Operations that propagate table-level metadata

* `copy`
* `getindex` with `!` or `:` as row selector and `:` as column selector
* `DataFrame`

### Operations that preserve column-level metadata

* `select!` and `transform!` if a single operation is passed  selector
  that selects one or multiple columns
* `leftjoin!` for the left table columns
* `insertcols!` for existing columns
* `setindex!` for all columns that are not changed or if it is assigning data
  frame to a data frame with all rows selected
* broadcasted assignment for all columns that are not changed

### Operations that propagate column-level metadata

* `copy`
* `getindex` with `!` or `:` as row selector
* `select`, `transform`, and `combine` if a single operation is passed  selector
  that selects one or multiple columns
* `DataFrame`
* `hcat`