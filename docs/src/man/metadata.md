# Metadata

## Design of metadata support

DataFrames.jl allows you to store and retrieve metadata on table and column
level. This is supported using the functions defined by DataAPI.jl interface
`hasmetadata` and `metadata`. These functions work with `DataFrame`,
`SubDataFrame`, `DataFrameRow`, `GroupedDataFrame`, `DataFrameRows`, and
`DataFrameColumns` objects.

Assume that we work with a data frame `df` that has a column `:col` at position
`2`.

### Contract for `hasmetadata`

In DataFrames.jl context `hasmetadata(df)` can return either `true` or `false`.
If `false` is returned this means that data frame `df` does not have any table
level metadata defined. If `true` is returned it means that at some table level
metadata is defined for `df`. Similarly `hasmetadata(df, :col)`,
`hasmetadata(df, "col")`, or `hasmetadata(df, 2)` can be used to check if column
`:col` has some metadata attached to it.

Although `hasmetadata` is guaranteed to return `Bool` value in DataFrames.jl
it is recommended to check its return value against `true` and `false` explicitly
using the `===` operator. The reason is that, in generic code, `hasmetadata`
is also allowed to return `nothing` if the queried object does not support
attaching metadata.

### Contract for `metadata`

In DataFrames.jl `metadata(df)` always returns `Dict{String, Any}` storing
key-value mappings of table level metadata. To add or update metadata to a data
frame mutate the returned dictionary. An identical rule applies to
`metadata(df, :cols)`, `metadata(df, "cols")`, `metadata(df, 2)` for the
column level metadata.

Note that it is recommended to use strings as values of the metadata. The reason
is that some storage formats, like for example Apache Arrow, only support
storing string data as metadata values.

The `metadata` function called on objects defined in DataFrames.jl is not thread
safe and should not be used in multi-threaded code.
In particular, as an implementation detail, the first time `metadata` is called
on a data frame object, it will mutate it (this might change in the future
versions of DataFrames.jl).

In generic code it is recommended to run `hasmetadata` check before calling
`metadata` function. The reason is, that if some object does not support
metadata the call to `metadata` function throws an error.

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

An important design feature of handling metatada is how it is handled
when you transform data frames.

Here are the rules that currently are followed by DataFrames.jl.

!!! note

    The provided rules might change in the future. Any change to metadata
    propagation rules will not be considered to be a breaking change
    in DataFrames.jl and can be done in any minor release of DataFrames.jl.
    Such changes might be made based on users' feedback about what metadata
    propagation rules are most convenient in practice.

### Operations that preserve metadata

### Operations that propagate metadata

### Operations on several tables

### Operations that drop metadata
