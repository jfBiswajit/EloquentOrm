Source: https://laracasts.com/discuss/channels/eloquent/eloquent-sync-associate?page=1

#### hasOne / hasMany (1-1, 1-n), morphOne / morphMany (polymorphic 1-n)

- `save` (new or existing child)
- `saveMany` (array of models new or existing)
- `create` (array of attributes)
- `createMany` (array of arrays of attributes)

#### belongsTo (n-1, 1-1), morphTo (polymorphic n-1)

- `associate` (existing model)

#### belongsToMany (n-n), morphedToMany / morphedByMany (polymorphic n-n)

- `save` (new or existing model, array of pivot data, touch parent = true)
- `saveMany` (array of new or existing model, array of arrays with pivot data)
- `create` (attributes, array of pivot data, touch parent = true)
- `createMany` (array of arrays of attributes, array of arrays with pivot data)
- `attach` (existing model / id, array of pivot data, touch parent = true)
- `sync` (array of ids OR ids as keys and array of pivot data as values, detach = true)
- `updateExistingPivot` (relatedId, array of pivot data, touch)