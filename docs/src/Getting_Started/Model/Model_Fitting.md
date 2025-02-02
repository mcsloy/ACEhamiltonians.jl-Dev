# Model Fitting
The `ACEhamiltonians` framework offers a variety of fitting subroutines which are capable of anything from a single basis to an entire model.
When using the model-level automated fitting procedure, one need only specify the model to be fitted and the location(s) from which fitting data can be gathered.
Model-level fitting can be carried out with only a few lines of code, as evident from the following example. 
```julia
# Path to the database within which the fitting data is stored
database_path = "example_data.h5"

# Names/paths of/to the system(s) to which the model should draw fitting data
target_systems = ["0224"]

# Open up the HDF5 database within which the target data is stored
h5open(database_path) do database

    # Load the target system(s) for fitting
    systems = [database[target_system] for target_system in target_systems]

    # Perform the fitting operation
    fit!(model, systems; recentre=true)
end

# Don't forget to save the model after fitting (if needed)
```
This particular version of `fit!` requires only two positional arguments, but accepts up to six optional keyword arguments.
The first positional argument is the `Model` to be fitted and the second is a vector of HDF5 `Group` object(s) from which the fitting data is to be drawn.
It is important to note that the provided HDF5 groups must follow the structure specification provided [here](../Data/Database_Structure.md).
The optional keyword arguments made available are:

- `on_site_filter::Function`: the `DataSet` entities for all on-site bases will be passed through this filter function prior to fitting. This function should take as `DataSet` as its only argument and yield a, possibly filtered, `DataSet` as its return. If not specified this will default to `identity`.
- `off_site_filter::Function`: off-site equivalent to the `on_site_filter` argument.
- `tolerance::AbstractFloat`: only sub-blocks with least one element greater than or equal to `tolerance` will be loaded. This argument permits sparse blocks to be ignored during loading and fitting. Warning, this can negatively impact the performance of the off-site bases if the bond cutoff distance was not chosen with care and is therefore disabled by default. 
- `recentre::Bool`: By default, atoms loaded from the database are assumed to span the fractional coordinate domain of \[0.0, 1.0). Setting `recentre` to `true` will remap atomic positions to the fractional coordinate domain of \[-0.5, 0.5). This is used to ensure that atomic coordinates are consistent with the geometry layout used internally by FHI-aims. This is disabled by default and should only be used when loading real-space matrices generated by FHI-aims. 
- `refit::Bool`: By default already fitted bases will not be refitted, but this behaviour can be suppressed by setting `refit=true`.
- `target::String`: a string indicating which matrix should be fitted. This may be either `H` or `S`. If unspecified then the model's `.label` field will be read and used.

### Manual Fitting

Alternatively, one may manually select the fitting data and place it into a dictionary keyed by basis id (`Dict(basis_id, DataSet)`). This is then provided to the fitting function along with the model like so `fit!(model, fitting_data_dictionary)`. Note that one need only provide data for the bases that one wishes to fit. 
