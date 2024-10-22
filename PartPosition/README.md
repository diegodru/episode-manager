# What potential actions do you identify as requiring a recalculation of positions in the Parts linked to the Episode?

The method used when a position is provided with a newly added part depends on whether the part is meant to go at the beginning, the end, or in between any already placed parts (similar to a linked list).

If the part is placed:

* at the beginning: Then the position of each part of the list must be shifted +1, then register the part with position 0.

* at the end: Then the part can be registered with the next value following the highest part position already registered.

* in between other parts: Then every other part with a higher or equal position value needs to be shifted +1.

# What API Request payload structure would you use to reliable place the Parts in the position they should end up on? Consider separate endpoints to add, delete and sort Parts.

### The Add Method:

### The Delete Method:

### The Sort Method:

NOTE: The sort method is assumed to allow the user to reposition parts by providing an ordered array rather than sort by executing a closure (or by providing an expression). This enables the actual sorting by analyzing content be done locally on the user's end to then send the result to the server.

To handle sorting with the developer experience in mind, the expected payload could include an array with the current position values (or the parts' composite ids) of the parts wished to sort in the desired order. The payload needn't to include all parts of an episode, only the parts wished to be repositioned. The server would be responsible to update the position field of the parts included in the array with the index in which its current position value (or the part's composite id) is found within the array.

Then, the remaining parts not included in the array (iterating from the lowest position value to the highest) would have their position field updated to a cached integer (with starting value equal to the length of the array) and increased by one with each successive update.

```JSON
{
  "episode": 3,
  "positions": [
    2,
    7,
    0,
    1
  ]
}
```

This approach would be stochastic as each call would with the same body would not yield the same order.

A deterministic approach could use the parts' identifiers, like so:

NOTE: __identifiers are represented as characters for visual distinction to positions__
```JSON
{
  "episode": 3,
  "parts": [
    'k',
    'a',
    'j',
    'c'
  ]
}
```

### Implement the calculation of the new positions for Parts when changes are performed that affect them.

Code examples are provided in the [index.php](./index.php) file.

# Do you have another approach on how this can be tackled efficient and reliably at scale?

## Queue a method for when a large requests come in, then notify the calling user when it is done and warn any other collaborating user who attempts to queue another method that one is currently being processed.

## Concurrency is always an option with divide and conquer.

## Create an index on the parts table if using a database.

## Attempt to insert at constant time O(1)

Worst case scenario, an insertion at the beginning would require to iterate over every part of an episode to update the position, making the time it takes to insert depend on the number of parts registered O(n). This could be costly if the system has episodes with a significant number of parts registered.

To reduce the potentially costly lapse to insert, the implementation could rely on the following:

### Fractional positioning indices

using floating point numbers to allow for fractional indexing on the position field could achieve constant time when inserting a new part in between. returning only an ordered array of part objects encapsulating the position field on the server side. using 32 bits floating point numbers for small systems and extending to 64 or even 128 bits for much larger systems.

After inserting several Parts between 2.0 and 3.0, the gaps may become too small to continue. In this case, you can re-normalize (or re-index) the order values occasionally by recalculating new, evenly spaced numbers.
re-normalizing is a relatively rare operation and is only necessary when you exhaust the available space between two order values

The implementation for the fractional indexing is provided below:

```
function insert($episodeId, $partId, $data) {
}

function delete($episodeId, $partId) {
}

function sort($episodeId, $partId) {
}
```
### Linked list structure with a map representation

Another approach for an attempt at constant time when inserting can include a linked list-like structure rather than registering positions. Returning an object instead of a list of parts where each key is the unique identifier to a map for the part data as its value which in part (no pun intended) has a key (i.e. "next") containing the id of the following part as its value. This way, the user can parse the structure as a hash map (dictionary) to easily iterate over the parts in the intended order. Although, this method would place the burden of sorting onto the user who would require to locally restructure the data for further efficient access. Moreover, this method would hinder performance for smaller systems as hashing identifiers would be more expensive than iterating over a small array.