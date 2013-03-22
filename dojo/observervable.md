QUESTION 1:

Now... is it just me, or Observable.js, between lines 71 and 126, could be made a _touch_ clear? I don't mean to criticise in _any_ way the amazing work you guys do; however, it does look like code that has been added to in the years, and it could be reorganised a little.

I personally think this is one of those cases where a _little_ bit of repetition would be a fair price to pay, in order to have code that is way easier to maintain. Plus, I've been looking at the code all afternoon, and cannot figure out why the code seems to delete the item twice (see writeup at the bottom of this email)

QUESTION 2:

Now, I still have a question about QueryEngine and JsonRest! Now that I have a (basic but half-decent) understanding if Observable, I can see what's going on. And... well, it looks to me like:

* JsonRest has a definite dataset returned once the server runs the query -- obviously.  * JsonRest effectively "gets" a queryEngine when it becomes cached with Cache() * In terms of JsonRest, what happens is that queryExecutor (what's returned by queryEngine) is called on the dataset to find out which position the item should go to (see writeup in this email)
  * This could well be a problem: for example, the sorting done by the database might well be different to the sorting done by queryEngine.
 * It just feels _strange_ to apply queryExecutor() on... a sub-set of the data, which happens to be already filtered and sorted
  * More problems: in a server query, I often implement things so that if I have `{ name: 't' }` in the filter, it will return anything starting with a `t`. This clashes with the `matches()` method in queryEngine which yes, allows you to use object.test(), but for strings it requires perfect matches. OK, it's not _actually_ a problem in this case, since matches will effectively always be used against *full* objects, but... it feels odd.
 * I read several times, in the doc, that when adding to JsonRest stores, you don't get to get the item in the 'right' spot for the lack of QueryEngine (and therefore queryExecutor). But! JsonRest Stores are cached a _lot_ of the time. So, the problem is not *actually* there -- but there is "oddity" for the points above...
  * [SOLVED] I wonder what happens with limits and counts (I still need to test this). That is: if I limit the search for records 20-40, but... WAIT! I just noticed that options.start and options.count are taken out of the options when applying queryEngine to get queryExecutor. Well, that "solves" it...!

Well... I am not sure anybody will have the stomach to read up to here...

THE WRITEUP

# QueryEngine

QueryEngine returns a function that, given a WHOLE array of data, is able to apply a filter to that data, and sort it. I guess it's structured this way so that you could define your own store Memory, but redefine the QueryEngine so that for example it matches *partial* items, and so on. This is actually very neat: the querying logic is 100% decoupled from the "query" itself. For convenience, Dojo provides a simple QueryEngine that does the usual (filter, sort) to the passed array. Easy.

The query returned also has a "matches()" method which is used to check if two items are the same -- it's the same "match" used by the filtering in query()

Note: QueryEngine doesn't actually make sense for JsonRest stores, because it's up to the server itself to do all the sorting/etc. In a JsonRest store, the query() function doesn't even call QueryEngine. If it did, it would apply filters/sorting on a *subset* of the data returned by the server.  Which would tend to be nonsense.

# Cache

Cache is a very neat module indeed: it takes a MasterStore and a CacheStore, and it *enriches* MasterStore's functions so that a lot of them will piggyback on CacheStore. For example, when you query() the MasterStore, the query is executed and returned, BUT ALSO the results are stored onto the CacheStore. MasterStore is often JsonRest and CacheStore is often Memory for obvious reasons.
So:

* query() -- the query is executed always on the master store, and the result is placed into the CacheStore.

* get() will get the result from the CacheStore if available -- if not, it will getch it from the MasterStore, and will then put() it in the CacheStore

* put()/add() will add to the MasterStore, and then will ALSO put the result into CacheStore

* remove() will delete from the MasterStore and then, if successful, it will delete from the CacheStore

* evict() will simply evict an item from the MasterStore.

Cache() also does something interesting: it sets the QueryEngine of the
Master store as either MasterStore's QueryEngine or CacheStore's QueryEngine, depending on which one is available: `queryEngine: masterStore.queryEngine || cachingStore.queryEngine,`. This means that, effectively, the JsonRest store _will_ end up with a QueryEngine.

===== DIDN'T I write, a little earlier, that QueryEngines didn't make much
sense for JsonRest stores?!? =====

Note: Cache() suffers from what I consider a bug: they will put, in the cache, a null value if the store returns null (which is the case when a JsonRestStore returns ""):   `cachingStore.add(typeof result == "object" ?  result : object, directives);`. Bug reported, I hope it gets looked at and fixed since the fix doesn't break anything!

# Observer

This is when things stop being easy and dandy. Whereas Cache() is a beauty of simplicity, and reading it makes you go "I understand Dojo! I do! I do!", you land on Observer.js and feel like you need a new brain.

The Observer() function changes the object passed; this is what it does: (This is from a developer's perspective):

* It adds a notify(object, existingId) function, which tells listeners what has changed
* Changes put(), add() and remove() so that, once they are done, they call notify() in the right way (put: notify( object, existingId), add: notify( object, null), remove: notify( null, existingId)
* It changes "query" so that it won't just return the original result: it will enrich the result with the observe() method, which can be used to trigger "listeners" when one of the elements returned by the query actually changes.

How does it achieve that? Well, here is how.

* It redefines the Store.query function, adding a whole lot to it
* QueryUpdaters is a variable that is store-wide. One store, one list of QueryUpdaters, one QueryUpdater(object, existingId) for each query() run.  (Well, actually, a QueryUpdater() function is only added to QueryUpdaters when the first listener is added for that particular query, but that's a "detail")
* Notify(object, existingId) calls every single QueryUpdater(object, existingId), one for each query with at least 1 observer.
* When calling query.observe( listener ), two things happen:
  * The listener(object, removedFrom, insertedInto) is added to a `listeners` array which is scoped in that particular query.
  *  FIRST TIME: a QueryUpdater(object, existingId) function is added to
QueryUpdaters, which is scoped in the Store as a whole. This specific, new QueryUpdater will check a changed object against the list of results of "its own" query
  * SIMPLIFICATION: a QueryUpdater( object, existingId), is called every time an object has changed, and it's like a kid in a candy shop, where candies are the query results: he needs to figure out if any of its own candies matches the one she was given, and when she has figured it out, she needs to tell every single listener about her discovery.

Then there is the second part of the function, which starts with:

    var removedObject, removedFrom = -1, insertedInto = -1;

Where its goal is to set these three variables, AND change resultArray() so that it's up-to-date (and therefore results.forEach() *after* a change will result in the updated information).

This is what happens:

*  First block of code. If existingId is set, it means that it's either a deletion or an update . If it's a deletion, delete it. If it's an update, only deletes it if it doesn't have a queryExecutor (otherwise, it won't be able to work out where to place it)

* Second block of code: if there IS a query executor.
  * At this point, the element has been *deleted* by "first block of code" above. It's a matter of re-inserting it. Here the QueryExecutor is used to: 1) Match the old object using queryExecutor.matches() or queryExecutor.query() 2) Work out where the element ends up (with an `array.indexOf(queryExecutor(resultsArray), changed);` where we basically "look" for the position of `changed` (the new element) within the filtered/sorted data set (`queryExecutor(resultsArray)` )
  * Something strange: even though "first block of code" deletes the item from resultArray if QueryExecutor is there or if the object is null (deletion), "second block of code" ALSO seems to delete it...?

* Third block of code: if there is NO query executor
  * At this point, the element has NOT been deleted by "first block of code" above. We don't have a queryExecutor. So, if it's a new element, it will just be placed at the end; and if it's an edit, it will be left alone where it is.

