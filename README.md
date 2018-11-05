# Content-Provider
How Content Providers Act<br>
<a href="https://github.com/ChanakaWeerasinghe/SendingApp">Sending App</a><br>
<a href="https://github.com/ChanakaWeerasinghe/Resiving">Reciving App</a><br>

11.1: Share Data Through Content Providers
Contents:
What is a Content Provider
What is a Content Resolver
Example of an app sharing data using a Content Provider
What Content Providers are good for
App Architecture with a Content Provider
Implementing a Content Provider
Data
Contract
Methods: insert, delete, update, query
query() method
Using a Content Resolver
Permissions for sharing
Related practical
Learn more


What is a Content Provider?
A ContentProvider is a component that interacts with a repository. The app doesn't need to know where or how the data is stored, formatted, or accessed.
A content provider:
Separates data from the app interface code
Provides a standard way of accessing the data
Makes it possible for apps to share data with other apps
Is agnostic to the repository, which could by a database, a file system, or the cloud.

What is a Content Resolver?
To get data and interact with a content provider, an app uses a ContentResolver to send requests to the content provider.
The ContentResolver object provides query(), insert(), update(), and delete() methods for accessing data from a content provider.
Each request consists of a URI and a SQL-like query, and the response is a Cursor object.
Note: You learned about cursors in the Data Storage chapters, and there is a recap later in this chapter.
The following diagram shows the query flow from an activity using a content resolver to the content provider to data in a SQL database, and back. Note that storing the data in a SQLite database is common, but not required. Query progression through app component

Example of an app sharing data using a Content Provider
Consider an app that keeps an inventory of hats and makes it available to other apps that want to sell hats. The app that owns the data manages the inventory, but does not have a customer-facing interface. Two apps, one that sells red hats and one that sells fancy hats, access the inventory repository, and each fetch data relevant for their shopping apps. Diagram showing how other apps might  use a content provider.

What Content Providers are good for
Content providers are useful for apps that want to make data available to other apps.
With a content provider, you can allow multiple other apps to securely access, use, and modify a single data source that your app provides. Examples: Warehouse inventory for retail stores, game scores, or a collection of physics problems for colleges.
For access control, you can specify levels of permissions for your content provider, specifying how other apps can access the data. For example, stores may not be allowed to change the warehouse inventory data.
You can store data independently from the app, because the content provider sits between your user interface and your stored data. You can change how the data is stored without needing to change the user-facing code. For example, you can build a prototype of your shopping app using mock inventory data, then later replace it with an SQL database for the real data. You could even store some of your data in the cloud and some locally, and it would be all the same to your users.
Another benefit of separating data from the user interface with a content provider is that development teams can work independently on the user interface and data repository of your app. For larger, complex apps it is very common that the user interface and the data backend are developed by different teams, and they can even be separate apps; that is, it is not required that the app with the content provider have a user interface. For example, your inventory app could consist only of the data and the content provider.
There are other classes that expect to interact with a content provider. For example, you must have a content provider to use a loader, such as CursorLoader, to load data in the background. You will learn about loaders in the next chapter.
Note: If your app is the only one using the data, and you are developing all of it by yourself, you probably don't need a content provider.

App Architecture with a Content Provider
Architecturally, the content provider is a layer between the content-providing app's data storage backend and the rest of the app, separating the data and the interface.
To give you a picture of the whole content provider architecture, this section shows and summarizes all the parts of the implemented content provider architecture, as shown in the following diagram. Each part will be discussed in detail next. Content Provider architecture
Data and Open Helper: The data repository. The data could be in a database, a file, on the internet, generated dynamically, or even a mix of these. For example, if you had a dictionary app, the base dictionary could be stored in a SQLite database on the user's device. If a definition is not in the database, it could get fetched from the internet, and if that fails, too, the app could ask the user to provide a definition or check their spelling.
Data used with content providers is commonly stored in SQLite databases, and the content provider API mirrors this assumption.
Contract: The contract is a public class that exposes important information about the content provider to other apps. This usually includes the URI schemes, important constants, and the structure of the data that will be returned. For example, for the hat inventory app, the contract could expose the names of the columns that contain the price and name of a product, and the URI for retrieving an inventory item by part number.
Content Provider: The content provider extends the ContentProvider class and provides query(), insert(), update(), and delete() methods for accessing the data. In addition, it provides a public and secure interface to the data, so that other apps can access the data with the appropriate permissions. For example, to get inventory information from your app's database, the retail hat app would connect to the content provider, not to the database directly, as that is not permitted.
The app that owns the data specifies what permissions other apps need to have to work with the content provider. For example, if you have an app that provides inventory to retail stores, your app owns the data and determines the access permissions of other apps to the data. Permissions are specified in the Android Manifest.
Content Resolver: Content providers are always accessed through a content resolver. Think of the content resolver as a helper class that manages all the details of connecting to a content provider for you. Mirroring the content provider's API, the ContentResolver object provides you with query(), insert(), update(), and delete() methods for accessing data of a content provider. For example, to get get all the inventory items that are red hats, a hat store app would build a query for red hats, and use a content resolver to send that query to the content provider.

Implementing a Content Provider
Referring to the previous diagram, to implement a content provider you need:
Data, for example, in a database.
A way for accessing the data storage, for example, through an open helper for a database.
A declaration of your content provider in the Android Manifest to make it available to your own and other apps.
The subclass of ContentProvider that implements the query(), insert(), delete(), update(), count(), and getType() methods.
Public contract class that exposes the URI scheme, table names, MIME type, and important constants to other classes and apps. While this is not mandatory, without it, other apps cannot know how to access your content provider.
A content resolver to access the content provider using the appropriate methods and queries.
Let's take a look at each of these components.

Data
The data is often stored in a SQLite database, but this is not mandatory. Data could be stored in a file or file system, on the internet, or created dynamically. Or even a mix of these options. To the app, the content resolver the fetched data always as a Cursor object, as if it all came from the same source and in the same format.
The content provider might access the data directly in the case of files, or it might do so through a helper class. For example, apps typically use an open helper to interact with a SQLite database, and the content provider interacts with the open helper to get the data.
Typically, the data is presented to the content provider by the data store as tables, similar to database tables, where each row represents one entry, and each column represents an attribute for that entry. For example, each row contains one contact, and may have columns for email addresses and phone numbers. The structure of the tables is exposed in the contract.
Note: If you are just working with files, you can use the predefined FileProvider class.

Contract
The contract is a public class that exposes important information about an app's content provider so that other apps know how to access and use the content provider.
Using a contract separates public from private app information, design from implementation, and gives other apps one place to get all the information they need to work with a content provider. While the underlying app may change, a contract defines an API that ideally does not change once the app is published.
The contract for a content provider typically includes:
Content URI and URI scheme. The URI scheme shows how to build URIs to access the content provider's data. It's the API for the data.
Table constants. Makes table and column names available as constants, because they are needed to extract data from the returned cursor object.
MIME types, which have information on the data format, so that the app can appropriately process returned data. For example, that data could be encoded in JSON or HTML, or use a custom format.
Other shared constants that make it more convenient for an app to use the content provider.
Note: Contracts are not limited to content providers. You can use a contract anytime you want to share constants across classes of your app or make information about your app available to other apps.
URI Scheme & Content URI
Apps send requests to the content provider using Uniform Resource Identifiers or URIs.
A content URI for content providers has this general form:
scheme://authority/path/ID
scheme is always content:// for content URIs.
authority represents the domain, and for content providers customarily ends in .provider
path is the path to the data.
ID uniquely identifies the data set to search.
For example, the following URI could be used to request all the entries in the "words" table:
content://com.android.example.wordcontentprovider.provider/words
The URI scheme for a content provider is defined in the Contract so that it is available to any app that wants to query this content provider. Customarily, this is done defining constants for AUTHORITY, CONTENT_PATH, and CONTENT_URI.
AUTHORITY. Represents the domain. For content providers this includes the unique package name and ends in .provider
CONTENT_PATH. The content path is an abstract semantic identifier of the data you are interested in. It does not predict or presume in what form the data is stored or organized in the background. As such, the path could resolve in the name of a table, the name of a file, or the name of the list.
CONTENT_URI. This is a content:// style URI to one set of data. If you have multiple "data containers" in the backend, you would create a content URI for each. For example, you would create a content URI for each table that can be queried. Use the Uri helper class for building and manipulating URIs.
In code, this might look as follows:
public static final String AUTHORITY =
"com.android.example.minimalistcontentprovider.provider";

public static final String CONTENT_PATH =  "words";
public static final Uri CONTENT_URI = Uri.parse("content://" + AUTHORITY +
"/" + CONTENT_PATH);
Tables in the Contract
A common way of organizing a contract class is to put definitions that are global to your database into the root level of the class. Usually, this is the name of the database.
Create a static abstract inner class for each table with the column names. This inner class commonly implements the BaseColumns interface. By implementing the BaseColumns interface, your class can inherit a primary key field called _ID that some Android classes, such as cursor adapters, expect to exist. This is not required, but can help your database work harmoniously with the Android framework.
Your code in the contract might look like this:
public static final String DATABASE_NAME = "wordlist";

public static abstract class WordList implements BaseColumns {
    public static final String WORD_LIST_TABLE = "word_entries";
    // Column names...
    public static final String KEY_ID = "_id";
    public static final String KEY_WORD = "word"
}
MIME Type
The MIME type tells an app, what type and format received data is in, so that it can process the data appropriately. Common MIME types include text/html for web pages, and application/json. If your content provider returns data in either of those standard formats, you should use the standard MIME types. A full list of these standard types is available on the IANA MIME Media Types website.
However, your content provider probably returns data specific to your app. In that case, you will need to specify a custom MIME type.
For content URIs that point to a row or rows of table data, and that are thus unique to your app, the MIME type should be in Android's vendor-specific MIME format. The general format is:
type.subtype/provider-specific-part
Where the parts should be:
Type part: vnd
Subtype part:
If the URI pattern is for a single row: android.cursor.item/
If the URI pattern is for more than one row: android.cursor.dir/
Provider-specific part: vnd..
You supply the and .
The value should be globally unique. A good choice for is your company's name or some part of your application's Android package name.
The value should be unique to the corresponding URI pattern. A good choice for the is a string that identifies the table associated with the URI.
For example, if a provider's authority is com.example.app.provider, and it exposes a table named "words", the MIME type for multiple rows in "words" is:
vnd.android.cursor.dir/vnd.com.example.provider.words
For a single row of "words", the MIME type is:
vnd.android.cursor.item/vnd.com.example.provider.words
And you would specify it in your contract as:
static final String SINGLE_RECORD_MIME_TYPE =
        "vnd.android.cursor.item/vnd.com.example.provider.words";
static final String MULTIPLE_RECORDS_MIME_TYPE =
        "vnd.android.cursor.item/vnd.com.example.provider.words";
An app can call the getType() method of a content provider to find out what type of data to expect. Your getType() method might look like this:
@Override
public String getType(Uri uri) {
   switch (sUriMatcher.match(uri)) {
       case URI_ALL_ITEMS_CODE:
           return MULTIPLE_RECORDS_MIME_TYPE;
       case URI_ONE_ITEM_CODE:
           return SINGLE_RECORD_MIME_TYPE;
       default:
           return null;
   }
}
Read more about MIME types for content providers in the Android developer documentation.
Note: The MIME type does not tell the clients how to process that data. As such, the custom MIME type only provides a hint, and the contract should provide additional information to the client as to what the expected data formats are.

Methods: insert, delete, update, query
The content resolver makes available query, insert, delete, and update methods, and the content provider implements those same methods to access the data.
As such, it is a good practice to keep the names, method arguments, and return values consistent between all components. This makes implementation and maintenance a lot easier.
The following diagram shows the APIs between the conceptual building blocks of an app that uses a content provider to access data. Note in particular:
The methods are named the same and return the same data type throughout the stack (except for insert()).
Because it is common for a content provider to connect to a database, the query method returns a cursor. If your backend is not a database, the content provider must do the work of converting the returned data into the cursor format.
This diagram does not show additional helper classes, such as open helpers for databases, that may also use the same API convention. Use the same method signature for query - insert - delete - and update methods across all app components
When you create a content provider by extending the ContentProvider class, you need to implement the insert, delete, update, and query methods. If you follow the principle of making the method signatures the same across components, passing data back and forth does not require a lot of code.
Here is are sample methods in a content provider. Note that the content provider receives the values to insert in the correct type, as ContentValues, calls the database, and builds and returns the required Uri for the content resolver.
Insert method
/**
* Inserts one row.
*
* @param uri Uri for insertion.
* @param values Container for Column/Row key/value pairs.
* @return URI for the newly created entry.
*/
@Override
public Uri insert(Uri uri, ContentValues values) {
   long id = mDB.insert(values);
   return Uri.parse(CONTENT_URI + "/" + id);
}
Delete method
/**
* Deletes records(s) specified by selectionArgs.
*
* @param uri URI for deletion.
* @param selection Where clause.
* @param selectionArgs Where clause arguments.
* @return Number of records affected.
*/
@Override
public int delete(Uri uri, String selection, String[] selectionArgs) {
   return mDB.delete(parseInt(selectionArgs[0]));
}
Update method
/**
* Updates records(s) specified by selection/selectionArgs combo.
*
* @param uri URI for update.
* @param values Container for Column/Row key/value pairs.
* @param selection Where clause.
* @param selectionArgs Where clause arguments.
* @return Number of records affected.
*/
@Override
public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
   return mDB.update(parseInt(selectionArgs[0]), values.getAsString("word"));
}

query() method
The query method in the content provider has the following signature:
public Cursor query(Uri uri, String[] projection, String selection,String[] selectionArgs, String sortOrder){}
The arguments represent the parts of an SQL query and are discussed below. The query method of the content provider must parse the URI argument and determine the appropriate action.
URI Matching
It is a good practice to use an instance of the UriMatcher class to match the URIs. UriMatcher is a helper class for matching URIs for content providers.
Create a new UriMatcher in your content provider.
private static UriMatcher sUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
In onCreate() add the URIs to match the the matcher. These are the content URIs defined in the Contract. You may want to do this in a separate method, initializeUriMatching() that you call in onCreate(). The example code includes URIs requesting all items, one item by ID, and the item count.
/**
* Defines the accepted Uri schemes for this content provider.
* Calls addURI() for all of the content URI patterns that the provider should recognize.
*/
private void initializeUriMatching() {

   // Matches a URI that is just the authority + the path,
   // triggering the return of all words.
   sUriMatcher.addURI(AUTHORITY, CONTENT_PATH, URI_ALL_ITEMS_CODE);

   // Matches a URI that references one word in the list by its index.
   sUriMatcher.addURI(AUTHORITY, CONTENT_PATH + "/#", URI_ONE_ITEM_CODE);

   // Matches a URI that returns the number of rows in the table.
   sUriMatcher.addURI(AUTHORITY, CONTENT_PATH + "/" + COUNT, URI_COUNT_CODE);
}
The query method switches on the matching URI to query the database for all items, one item, or an item count, as shown in this example code.
@Override
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs,
                   String sortOrder) {

   Cursor cursor = null;

   // Determine integer code from the URI matcher and switch on it.
   switch (sUriMatcher.match(uri)) {
       case URI_ALL_ITEMS_CODE:
           cursor =  mDB.query(ALL_ITEMS);
           Log.d(TAG, "case all items " + cursor);
           break;
       case URI_ONE_ITEM_CODE:
           cursor =  mDB.query(parseInt(uri.getLastPathSegment()));
           Log.d(TAG, "case one item " + cursor);
           break;
       case URI_COUNT_CODE:
           cursor = mDB.count();
           Log.d(TAG, "case count " + cursor);
           break;
       case UriMatcher.NO_MATCH:
           // You should do some error handling here.
           Log.d(TAG, "NO MATCH FOR THIS URI IN SCHEME: " + uri);
           break;
       default:
           // You should do some error handling here.
           Log.d(TAG, "INVALID URI - URI NOT RECOGNIZED: "  + uri);
   }
   return cursor;
}

Using a Content Resolver
The ContentResolver object provides methods to query(), insert(), delete(), and update() data. Thus, the content resolver mirrors the content providers API and manages all interaction with the content provider for you. In most situations, you can use the default content resolver provided by the Android system.
Cursors
The content provider always presents the query results as a Cursor in a table format that resembles of a SQL database. This is independent of how the data is actually stored.
A cursor is a pointer into a row of structured data. You can think of it as a linked list of rows. The Cursor class provides methods for moving the cursor through that structure, and methods to get the data from the columns of each row.
When a method returns a cursor, you iterate over the result, extract the data, do something with the data, and finally close the cursor to release the memory.
If you use a SQL database, as shown above, you can implement your open helper to return a cursor, and then again, the content provider returns a cursor via the content resolver. If your data storage returns data in a different format, you will have to convert it into a cursor, usually a MatrixCursor.
The query() method
To make a query to the content provider:
Create an SQL-style query.
Use a content resolver to interact with the content provider to execute the query and return a Cursor.
Process the results in the Cursor.
The query method has the following signature:
public Cursor query(Uri uri, String[] projection, String selection,String[] selectionArgs, String sortOrder){}
The arguments to this method represent the parts of a SQL query. Even if you are using another kind of backend, you must still accept a query in this style and handle the arguments appropriately.
uri	
The complete content URI queried. This cannot be null. You get the information for the correct URI from the contract. For example:
String queryUri = Contract.CONTENT_URI.toString();
projection	
A string array with the names of the columns to return for each row. Setting this to null returns all columns. For example:
String[] projection = new String[] {Contract.CONTENT_PATH};
selection	
Indicates which rows/records of the objects you want to access. This is a WHERE clause excluding the actual where. For example:
String where = KEY_WORD + " LIKE ?";
selectionArgs	
Argument values for the selection criteria. If you include ?s in selection, they are replaced by values from selectionArgs, in the order that they appear. IMPORTANT: It is a best security practice to always separate selection and selectionArgs. For example:
String[]whereArgs = new String[]{searchString};
sortOrder	
The order in which to sort the results. Formatted as an SQL ORDER BY clause (excluding the ORDER BY keyword). Usually ASC or DESC; null requests the default sort order, which could be unordered.
And you make a query to to the content provider like this:
Cursor cursor = getContentResolver().query(Uri.parse(queryUri), projection, selectionClause, selectionArgs, sortOrder);
Note: The insert, delete, and update methods are provided for convenience and clarity. Technically, the query method could handle all requests, including those to insert, delete, and update data.

Permissions for sharing
By default, apps cannot access the data of other apps. Both apps involved in sharing data need to have permission to do so.
The content provider must allow other apps to access it's data.
The user must allow the client app to access the content provider.
Permissions in from the content provider
To make your content provider visible and available to other apps, you need to declare in the AndroidManifest of the provider.
<provider android:name=".WordListContentProvider" android:authorities="com.android.example.wordlistsqlwithcontentprovider.provider" android:exported="true"/>
The android:exported property makes it explicit that other apps can use this content provider.
With no permissions set explicitly, any other app can access the content provider for reading and writing. To limit and make explicit access constraints, set permissions inside the provider tag of the content provider, where myapp is the unique name of your app:
android:readPermission="com.android.example.wordlistsqlwithcontentpfonrovider.PERMISSION"
android:writePermission="com.android.example.wordlistsqlwithcontentprovider.PERMISSION"
The permission string should be unique to your content provider, so that it only grants privileges for your content provider.
While the string can be anything, using the package name guarantees uniqueness.
Permissions client app requests from user
In order to access the content provider, the client app needs to declare permissions in the Android Manifest for that content provider.
<uses-permission android:name = "com.android.example.wordlistsqlwithcontentprovider.PERMISSION"/>
Permissions are not covered in detail in these concepts.
You can learn more in Declaring Permissions, System Permissions, and Implementing Content Provider Permissions.

Related practical
The related practicals documentation is in Android Developer Fundamentals: Practicals.

Minimalist Content Providers
Add a Content Provider to WordListSQL
Sharing Content with Other Apps

Learn more
Developer Documentation:
Content Provider Basics
Content Providers
Uniform Resource Identifiers or URIs
MIME type
MatrixCursor and Cursors
Working with System Permissions
Implementing Content Provider Permissions
Videos:
Android Application Architecture
Android Application Architecture: The Next Billion Users
