# Observations on Dojo's observable


## The story from the user's point of view

Observable allows you to get a store, and make it possible to "observe" queries. This means that if the result set of a query changes, you will be notified.
Imagine you have two queries, for page1 and page2:

    page1 = store.query({}, { start: 0, count: 4 });
    when( page1 ).then( function( resultSet ){
      console.log( resultSet );
    });
    page2 = store.query({}, { start: 4, count: 4 });
    when( page2 ).then( function( resultSet ){
      console.log( resultSet );
    });

If the stores are Observable, you can do this:

    page1.observe( function( object, removedFrom, insertedInto ){
      // Party!
    });
    page1.observe( function( object, removedFrom, insertedInfo ){
      // Yes, you can observe twice, no worries
    });


The function passed as a parameter is called a _listener_. Note: the _listener_ will only ever be notified for changes made to _its_ specific result set (obviously).
The parameters `removedFrom` and `insertedInfo` take a while to get used to. Basically, there are three possibilities:

* An object is added: `listener( object, -1, X )` (where `X` is where the value will end up)
* An object is edited: `listener( object, X, Y)` (where `X` is where the object will disappear from, and `Y` is where it will appear)
* An object is deleted: `listener( object, X, -1 )` (where X is where the object will disappear from)

These are the three sane combinations of parameters and what they mean. They express exactly what could possibly happen to an object (added, moved, deleted).

There is more: a store will also have the `notify()` method. This will notify a store that an object has changed. Its signature is different compared to the listeners, because _`notify()` is store-wide, not query-wide_. This is crucial to keep in mind. This is how it works:

    store.notify( object, id )

There are three possibilities:

* `object` is set, `id` `undefined` => The `object` was added
* `object` is set, id is set => The `object` was edited
* `object` is `undefined`, id is set => The object `id` was deleted

So:

* `observe()` will trigger `listener` if an object within the result set has changed
* `notify()` will tell the store that an element has changed

## Behind the scenes

Observable will basically change a store so that:

* It adds to the store's editing functions, `put()`/`add()`, a `get()` or a `remove()`, so that they will trigger `store.notify()` as described above
* It adds to the store's `query()` function, so that it has the `observe()` function added to it
* It adds a store-wide `queryUpdaters` array, where each element (each `queryUpdater`) has the same signature as `notify()`. Note: each query _might_ add _one_ `queryUpdater` entry to this array. I say _might_ because a query without observers will not add an element to `queryUpdaters`.
* It adds a query-wide `listeners` array

So, writing:

    page1 = store.query({}, { start: 0, count: 4 });

At this point:

* The store-wide array queryUpdaters has zero elements. 
* `Page1`'s `listeners` array has zero elements.

Then:

    page1.observe( function( object, removedFrom, insertedInto ){
      // Party!
    });

At this point:

* The store-wide array queryUpdaters has one element (for this query)
* `Page1`'s `listeners` array has one element (the function passed)

Then:

    page1.observe( function( object, removedFrom, insertedInto ){
      // Party again!
    });

* The store-wide array queryUpdaters _still_ has one element (the one for this query)
* `Page1`'s `listeners` array has two elements

    page2 = store.query({}, { start: 4, count: 4 });

* Nothing changes observer-wise.

Then:

    page2.observe( function( object, removedFrom, insertedInto ){
      // Party!
    });

* The store-wide array queryUpdaters has _two_ elements (one for each query with an observer)
* `Page2`'s `listeners` array has one element

To sum it up:


    page1 = store.query({}, { start: 0, count: 4 });

    page1.observe( function( object, removedFrom, insertedInto ){
      // Party!
    });
    page1.observe( function( object, removedFrom, insertedInto ){
      // Party again!
    });

    page2 = store.query({}, { start: 4, count: 4 });
    page2.observe( function( object, removedFrom, insertedInto ){
      // Party!
    });

* The store-wide array queryUpdaters has _two_ elements (one for each query with an observer)
* `Page2`'s `listeners` array has one element

## Notify and queryUpdaters

The real heart of Observable is in the `queryUpdaters` and how `notify()` uses them.
When you call:

    store.notify(object, id)

You are actually calling every `queryUpdater` function for that store: `queryUpdater(object, id)`. Remember that there is one `queryUpdater` per query (unless a query doesn't have observers, in which case the `queryUpdater` won't be there).

So, every time you run `store.notify(object, id)`, you are telling _every_ query made in the store that the `object` with id `id` has changed.


### The hard work of the `queryUpdater`

The `queryUpdater` has a very tough task. It has:

* The results set for that particular query. This is likely to be a sub set due to filtering, `count`, `limit`, and so on
* The object that was changed (which could have been added, etc.)
* The query object itself

And it needs to:

* Check if that query's results set *would* indeed *contain* the new element. IF SO:
 * Change the resultSet to reflect the change in the query's result
 * Call that specific query's listeners to tell them that the query has indeed changed

Note that it is indeed a pretty hard task, if you consider that:

* You may not have access to the _complete_ result set AND:
* The item was modified, and it no longer "fits" into the query results. For example, the query is sorted by name, you are viewing the first page which has names up to `M`, and you changed the name into `Zebra`

Consider that the `queryUpdater` will be called for every query run, and each query may or may not end up containing the element.

In terms of accessing the complete result set, IF you are using a local store (that is, a store that is basically a long Array and a queryEngine function that allows you to sort/match items), then you are in luck: it's pretty simple to figure out if an item would indeed be in a specific query. However, if your store is a JsonRest one, then:

* You do not have access to the full result set
* The querying/sorting is done by the _server_. So, you are completely 100% out of luck trying to figure out whether your queries are meant to include a new element


## Why JsonRest and Observable don't work well together

Observable.js really, really assumes that a queryEngine is there (from which queryExecutor will be extracted). JsonRest stores don't have a queryEngine -- in fact, Writing a queryEngine for a JsonRest store is hardly possible, since the queryEngine is meant to filter/sort/etc. a bunch of _local_ data, whereas a JsonRest store has no idea about the whole lot of data, nor about sorting etc. (the database engine will do that).

Since JsonRest stores are _very_ often cached by Cache, `Observable` ends up using the Cache store's `queryEngine`/`queryExecutor`, which turns things "interesting":

* JsonRest has a definite dataset returned once the server runs the query -- obviously.
* In terms of JsonRest, the queryExecutor (from Cache.js) is called on the dataset to find out which position the item should go to
  * What if the server sorts data in a different way...?
  * We are actually running queryExecutor() on... a _sub-set_ of the data, which happens to be already filtered and sorted; this is not what `queryExecutor` is created for.
  * What if the server's filtering functions differ from the queryEngine's? For example, for the server filtering by `{ name: 't' }` might well mean "everything starting with 't', which would clash with the `matches()` method in queryEngine. The _whole_ object is passed as a filter by Observable, but even then the server might filter upper/lowercase differently.


### QueryEngine

QueryEngine returns a function that, given a WHOLE array of data, is able to apply a filter to that data, and sort it. I guess it's structured this way so that you could define your own store Memory, but redefine the QueryEngine so that for example it matches *partial* items, and so on. This is actually very neat: the querying logic is 100% decoupled from the "query" itself. For convenience, Dojo provides a simple QueryEngine that does the usual (filter, sort) to the passed array. Easy.

The query returned also has a "matches()" method which is used to check if two items are the same -- it's the same "match" used by the filtering in query()

Note: QueryEngine doesn't actually make sense for JsonRest stores, because it's up to the server itself to do all the sorting/etc. In a JsonRest store, the query() function doesn't even call QueryEngine. If it did, it would apply filters/sorting on a *subset* of the data returned by the server.  Which would tend to be nonsense.

### Cache

Cache is a very neat module indeed: it takes a MasterStore and a CacheStore, and it *enriches* MasterStore's functions so that a lot of them will piggyback on CacheStore. For example, when you query() the MasterStore, the query is executed and returned, BUT ALSO the results are stored onto the CacheStore. MasterStore is often JsonRest and CacheStore is often Memory for obvious reasons.
So:

* query() -- the query is executed always on the master store, and the result is placed into the CacheStore.

* get() will get the result from the CacheStore if available -- if not, it will getch it from the MasterStore, and will then put() it in the CacheStore

* put()/add() will add to the MasterStore, and then will ALSO put the result into CacheStore

* remove() will delete from the MasterStore and then, if successful, it will delete from the CacheStore

* evict() will simply evict an item from the MasterStore.

Cache() also does something interesting: it sets the QueryEngine of the
Master store as either MasterStore's QueryEngine or CacheStore's QueryEngine, depending on which one is available: `queryEngine: masterStore.queryEngine || cachingStore.queryEngine,`. This means that, effectively, the JsonRest store _will_ end up with a QueryEngine -- one meant to be used on data.


