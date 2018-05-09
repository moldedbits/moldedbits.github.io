---
layout: post
title:  "Android Paging Library with Content Providers"
date:   2018-05-02 8:00:00
author: anuj
categories: Android
---
Android Architecture Components recently introduced the [Paging Library][1].The paging library makes it easier for your app to gradually load information as needed from a data source, without overloading the device or waiting too long for a big database query.

The examples provided with the library show the ease with which it can be used with Room. In this post I will show how easily it integrates with ContentProviders as well.

##### Behind the scenes

Paging library is smart about where it performs the IO. The threading model it uses is

![Paging Threading][paging-threading]{:style="width: 500px; margin:auto;"}

##### Lets get started!

For this demo, we will build an app that displays the list of user's contacts in a `RecyclerView`.

The first thing we'll need to implement is a custom DataSource. There are three options the library provides us,

* `PageKeyedDataSource` - If you have the links to the next and previous pages
* `ItemKeyedDataSource` - if you need to use data from item `N-1` to load item `N`
* `PositionalDataSource` - if you can load pages of a requested size at arbitrary positions, and provide a fixed item count

For our use case, as we will be using `Queries`, PositionalDataSource suits best.

We need to implement 2 methods in our custom data source,

```kotlin
override fun loadInitial(params: LoadInitialParams, callback: LoadInitialCallback<Contact>) {
    callback.onResult(getContacts(params.requestedLoadSize, params.requestedStartPosition), 0)
}

override fun loadRange(params: LoadRangeParams, callback: LoadRangeCallback<Contact>) {
    callback.onResult(getContacts(params.loadSize, params.startPosition))
}
```

For the `getContacts` implementation, we will use `ContentResolver`, which provides us access to the content model.

```kotlin
private fun getContacts(limit: Int, offset: Int): MutableList<Contact> {
    // Get the cursor
    val cursor = contentResolver.query(ContactsContract.Contacts.CONTENT_URI,
            PROJECTION,
            null,
            null,
            ContactsContract.Contacts.DISPLAY_NAME_PRIMARY +
                    " ASC LIMIT " + limit + " OFFSET " + offset)

    // load data from cursor into a list
    cursor.moveToFirst()
    val contacts: MutableList<Contact> = mutableListOf()
    while (!cursor.isAfterLast) {
        val id = cursor.getLong(cursor.getColumnIndex(PROJECTION[0]))
        val lookupKey = cursor.getString(cursor.getColumnIndex(PROJECTION[0]))
        val name = cursor.getString(cursor.getColumnIndex(PROJECTION[2]))
        contacts.add(Contact(id, lookupKey, name))
        cursor.moveToNext()
    }
    cursor.close()

    // return the list of results
    return contacts
}
```

And then we can create a `DataSource.Factory` which just returns an instance of our DataSource class.

In our ViewModel, we will create the LiveData,

```kotlin
lateinit var contactsList: LiveData<PagedList<Contact>>

fun loadContacts() {
    val config = PagedList.Config.Builder()
            .setPageSize(20)
            .setEnablePlaceholders(false)
            .build()
    contactsList = LivePagedListBuilder<Int, Contact>(
            ContactsDataSourceFactory(contentResolver), config).build()
}
```

and finally, we observe this in our fragment / activity

```kotlin
viewModel.loadContacts()
viewModel.contactsList.observe(this, Observer {
    adapter.submitList(it)
})
```

That's all there is to it. With this, we have a RecyclerView that efficiently loads data in chunks.

You can find complete source code on [github here](https://github.com/anujmiddha/paged-list-demo)

If you have any suggestions, let me know in comments!

Happy Coding!

[1]: https://developer.android.com/topic/libraries/architecture/paging
[paging-threading]: {{site.url}}/assets/images/paging-threading.gif "Paging Threading"
