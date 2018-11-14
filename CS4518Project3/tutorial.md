
# Antony Qin's Project3 Part2 Tutorial
## Overview of Tutorial
Duration: ???

This tutorial will show you how to build sqllite support into project2. And so, on app exit, the app remembers the images displayed and the order they were displayed in by storing the data in a database.

* Sqllite!

Prerequisites:

* Part2 of (my) CS4518 Project2
* Part1 of CS4518 Project3
* Knowledge of XML layout design
* File/Image loading

## The Main Code:
Duration: ??? minutes

### LogContract.java
* Go to LogContract.java
* Update SQL_CREATE_ENTRIES so that the created database has space to hold a picture's URL and order
* Add a new variable QUERY_PICS as it will be used as a SQL command in order to grab all the rows (image objects) in the database
    * I've created a class called FlickrImage.java that has respective getters/setters of column values
* Add the QUERY_ID_EXISTS as it will be used as a SQL command in order to check whether or not an image object (FlickrImage) already exists in the database

``` java
private static final String SQL_CREATE_ENTRIES =
   "CREATE TABLE " + LogEntry.TABLE_NAME +
   " (" +
   LogEntry._ID + " INTEGER PRIMARY KEY," +
   LogEntry.COLUMN_NAME_ENTRY + " TEXT," +
   LogEntry.COLUMN_NAME_URL + " TEXT," +
   LogEntry.COLUMN_NAME_ID + " TEXT," +
   LogEntry.COLUMN_NAME_PICORDER + " INTEGER" +
   ")";

public static final String QUERY_PICS = "SELECT " + "*" + " FROM " + TABLE_NAME;

public static final String QUERY_ID_EXISTS = "SELECT " +  COLUMN_NAME_ID + " FROM " + TABLE_NAME + " WHERE "  + COLUMN_NAME_ID + " = ";
```

* Also add the following method getUpdateOrderSQL in order to update the database (the picture's order position) from the main application:

``` java
public static String getUpdateOrderSQL(String id, int order)
{
   String update = "UPDATE " + TABLE_NAME + " SET " + COLUMN_NAME_PICORDER + " = " + order;
   update += " WHERE " + COLUMN_NAME_ID + " = " + id;

   return update;
}
```

### LogEntryDBHelper.java
* Now it's time to update LogEntryDBHelper.java
* We will need to add 3 methods to utilize the new functionality we put in LagContract.java
* Add a method called doesPicIDExists to check whether or not a picture is already in the database

``` java
public boolean doesPicIDExists(String id)
{
   String query = LogContract.LogEntry.QUERY_ID_EXISTS + id;
   Log.d(TAG, "doesPicIDExists() : QUERY IS - " + query);
   Cursor cursor = this.getReadableDatabase().rawQuery(query, null);

   if(cursor == null) return false;
   else
   {
      if(cursor.getCount() > 0) return true;
      else return false;
   }
}
```

* Add a method called updatePicOrderByID to update a picture's order position in the database

``` java
public void updatePicOrderByID(String id, int order)
{
   String update = LogContract.LogEntry.getUpdateOrderSQL(id, order);
   Log.d(TAG, "updatePicOrder() : UPDATE IS - " + update);
   this.getWritableDatabase().execSQL(update);
}
```

* Add a method called getPics return all the image objects (FlickrImage) in the database

``` java
public List<FlickrImage> getPics()
{
   List<FlickrImage> pics = new ArrayList<FlickrImage>();

   Cursor cursor = this.getReadableDatabase().rawQuery(LogContract.LogEntry.QUERY_PICS, null);
   try
   {
      cursor.moveToFirst();
      while (!cursor.isAfterLast())
      {
         FlickrImage img = new FlickrImage();

         img.setName(cursor.getString(cursor.getColumnIndex(LogContract.LogEntry.COLUMN_NAME_ENTRY)));
         img.setId(cursor.getString(cursor.getColumnIndex(LogContract.LogEntry.COLUMN_NAME_ID)));
         img.setURL(cursor.getString(cursor.getColumnIndex(LogContract.LogEntry.COLUMN_NAME_URL)));
         img.setOrder(cursor.getInt(cursor.getColumnIndex(LogContract.LogEntry.COLUMN_NAME_PICORDER)));

         pics.add(img);

         cursor.moveToNext();
      }
   }
   finally
   {
      cursor.close();
   }

   return pics;
}
```

### MainActivity.java
* Add a method called putFlickrImageToDB that takes an image object (FlickrImage) and puts its data as a row into the database you've declared (as db in my case)

``` java
private void putFlickrImageToDB(FlickrImage img)
{
   if(db.doesPicIDExists(img.getId()) == true) return; // Don't add row to database if URL of picture already exists

   ContentValues cv = new ContentValues();
   cv.put(LogContract.LogEntry.COLUMN_NAME_ENTRY, img.getName());
   cv.put(LogContract.LogEntry.COLUMN_NAME_URL, img.getURL());
   cv.put(LogContract.LogEntry.COLUMN_NAME_ID, img.getId());
   cv.put(LogContract.LogEntry.COLUMN_NAME_PICORDER, img.getOrder());

   db.getWritableDatabase().insert(LogContract.LogEntry.TABLE_NAME, null, cv);
}
```

* Whenever you create an image object (FlickrImage) in your code, call putFlickrImageToDB to ensure that the database remembers all the pictures in your application

``` java
// Add images to database
putFlickrImageToDB(mFlickrImages.get(i));
```

* Add calls to the updatePicOrderByID method of LogEntryDBHelper whereever in your code that you're swapping picture order around
    * In my case, mFlickrImages hold a list of image objects (FlickrImage) and mImageViewPos holds the current picture in each position, which may reflect whatever you're trying to do to hold/swap pictures on the screen

``` java
// Update picture order in database
db.updatePicOrderByID(mFlickrImages.get(mImageViewPos.get(0)).getId(), 0);
db.updatePicOrderByID(mFlickrImages.get(mImageViewPos.get(iv_index)).getId(), iv_index);
```

* Finally add a SharedPreference that remembers whether or not the pictures have been ordered (and therefore need to be extracted from the database and displayed again) the last time the app was open

``` java
final String PREFS_NAME = "Preferences";

// ...
SharedPreferences settings = getSharedPreferences(PREFS_NAME, 0);
if (settings.getBoolean("orderedAlready", false) == true) // Pics have been ordered
{
   mFlickrImages = db.getPics(); // Get all pictures put into database
   // Put the code here to parse that list and reorder/display your images as needed
}
// ...

// ...
settings.edit().putBoolean("orderedAlready", true).apply(); // Put this statement somewhere after your pictures have been ordered
// ...
```
    
### Congratulations, You've Completed the Tutorial!









