# Journal-C

### Level 1

Students will build a simple Journal app to practice MVC separation, protocols, master-detail interfaces, table views, and persistence. 

Journal is an excellent app to practice basic Cocoa Touch princples and design patterns. Students are encouraged to repeat building Journal regularly until the principles and patterns are internalized and the student can build Journal without a guide.

Students who complete this project independently are able to:

### Part One - Model Objects and Controllers

* understand basic model-view-controller design and implementation
* create a custom model object with a memberwise initializer
* understand, create, and use a shared instance
* create a model object controller with create, read, update, delete methods
* implement the Equatable protocol

### Part Two - User Interface

* implement a master-detail interface
* implement the UITableViewDataSource protocol
* understand and implement the UITextFieldDelegate protocol to dismiss the keyboard
* create relationship segues in Storyboards
* understand, use, and implement the 'updateWith' pattern
* implement 'prepareForSegue' to configure destination view controllers

### Part Three - Controller Implementation

* add data persistence using NSUserDefaults
* understand, use, and implement the builder methods pattern
* create a custom model object with a failable initializer
* use the array map method to translate objects

## Part One - Model Objects and Controllers

### Entry

Create an ```Entry``` model class that will hold a title, text, and timestamp for each entry.

1. Add a new ```Entry``` class as an ```NSObject``` subclass
2. Add properties for timestamp, title, and body text
3. Add a memberwise initializer that takes parameters for each property
    * note: Consider setting a default parameter value for timestamp.

### EntryController

Create a model object controller called ```EntryController``` that will manage adding, reading, updating, and removing entries. We will follow the shared instance design pattern because we want one consistent source of truth for our entry objects that are held on the controller.

1. Add a new ```EntryController``` class as an ```NSObject``` subclass
2. Add an entries NSArray property
3. Create a ```- (void)addEntry:(Entry *)entry``` method that adds the entry parameter to the entries array
4. Create a ```- (void)removeEntry:(Entry *)entry``` method that removes the entry from the entries array
    * note: Look at the ```NSArray``` documentation to find how to remove objects
5. Create a sharedController property as a shared instance. 
    * note: Review the syntax for creating shared instance properties

```
+ (EntryController *)sharedInstance {
    static EntryController *sharedInstance = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedInstance = [EntryController new];
    });
    return sharedInstance;
}
```

### Black Diamonds

* Implement the NSCoding protocol on the Entry class
* Create a Unit test that verifies NSCoding methodality by converting an instance to and from NSData

## Part Two - User Interface

### Master List View

Build a view that lists all journal entries. You will use a UITableViewController and implement the UITableViewDataSource methods.

The UITableViewController subclass template comes with a lot of boilerplate and commented code. For readability, please remove all unused boilerplate from your code. 

You will want this view to reload the table view each time it appears in order to display newly created entries.

1. Add a UITableViewController as your root view controller in Main.storyboard and embed it into a UINavigationController
2. Create an EntryListTableViewController file as a subclass of UITableViewController and set the class of your root view controller scene
3. Implement the UITableViewDataSource methods using the EntryController entries array
    * note: Pay attention to your ```reuseIdentifier``` in the Storyboard scene and your dequeue method call
4. Set up your cells to display the title of the entry
5. Implement the UITableViewDataSource ```commitEditingStyle``` methods to enable swipe to delete methodality
6. Add a UIBarButtonItem to the UINavigationBar with the plus symbol
    * note: Select 'Add' in the System Item menu from the Identity Inspector to set the button as a plus symbol, these are system bar button items, and include localization and other benefits

### Detail View

Build a view that provides editing and view methodality for a single entry. You will use a UITextField to capture the title, a UITextView to capture the body, a UIButton to save, and a UIButton to clear the title and body text areas.

Your Detail View should follow the 'updateWith' pattern for updating the view elements with the details of a model object. To follow this pattern, the developer adds an 'updateWith' method that takes a model object. The method updates the view with details from the model object.

1. Add an 'updateWith' method that takes a model object as a parameter (in this case, an Entry)
2. Implement the 'updateWith' methods to update all view elements that reflect details about the model object (in this case, the titleTextField and bodyTextView)

This view needs to serve as a reading and editing view. You will add a UITextField to display the title (titleTextField) and a UITextView to display the body text (bodyTextView), a 'Clear' button that resets both fields, and a 'Save' button that saves the new or changed entry.

#### View Setup

1. Add an ```EntryDetailViewController``` file as a subclass of UIViewController
2. Add a UIViewController scene to Main.storyboard and set the class to ```EntryDetailViewController```
3. Add a UITextField for the entry's title text to the top of the scene, add an outlet to the class file, and set the delegate relationship
4. Implement the delegate methods ```textFieldShouldReturn``` to resign first responder to dismiss the keyboard
5. Add a UITextView for the entry's body text beneath the title text field, add an outlet to the class file
6. Add a UIButton beneath the body text view, add an action to the class file that clears the text in the titleTextField and bodyTextView
7. Add a UIBarButtonItem to the UINavigationBar as a Save System Item, add an action to the class file
8. Add an Entry property ```@property (strong, nonatomic) Entry *entry```
9. In the action check if the entry property holds an entry, if so, update the properties of the entry. If not, call the save method on the ```EntryController```. After saving the entry, dismiss the current view.
    * note: You may need to add a segue to see a UINavigationBar on the detail view, and a UINavigationItem to add the UIBarButtonItem to the UINavigationBar, this will be covered in the next step

### Segue

You will add two separate segues from the List View to the Detail View. The segue from the plus button will tell the EntryDetailViewController that it should create a new entry. The segue from a selected cell will tell the EntryDetailViewController that it should display a previously created entry, and save any changes to the same.

1. Add a 'show' segue from the Add button to the EntryDetailViewController scene and give the segue an identifier
    * note: Consider that this segue will be used to add an entry when naming the identifier
2. Add a 'show' segue from the table view cell to the EntryDetailViewController scene and give the segue an identifier
    * note: Consider that this segue will be used to edit an entry when naming the identifier
3. Add a ```prepareForSegue``` method to the EntryListTableViewController
4. Implement the ```prepareForSegue``` method. Check the identifier of the segue parameter, if the identifier is 'toAddEntry' then we will present the destination view controller without passing an entry
5. Continue implementing the ```prepareForSegue``` method. If the identifier is 'toViewEntry' we will pass the selected entry to the DetailViewController by calling our ```- (void)updateWithEntry:(Entry *)entry``` method
    * note: You will need to capture the selected entry by using the indexPath of the selected cell
    * note: Remember that the ```- (void)updateWithEntry:(Entry *)entry``` method will update the destination view controller with the entry details

### Black Diamonds

* Implement UITableViewCellEditingStyles to enable swipe to delete entries on the List View
* Update Unit and UITests to verify delete methodality


## Part Three - Controller Implementation

You will use NSUserDefaults to add basic data persistence to the Journal app. 

### Add Factory Functions to Entry

Because ```Entry``` class objects are not plist compatible, and ```NSUserDefaults``` will only store classes that are, we have two options:

1. Implement NSCoding to allow native saving and loading from plist objects like NSUserDefaults
2. Add factory methods to make NSDictionary representations of the object and initialize new objects with a NSDictionary

There are pros and cons to both approaches. We've opted to go with the latter because it is closer to working with network services and APIs later on in the class.

1. Write a ```dictionaryCopy``` method that returns an ```NSDictionary``` with keys and values matching the properties of the object.
    * note: Avoid using 'Magic Strings' in your code. Create private string keys for the class for each property that will be stored to and pulled from a NSDictionary (ex. ```static NSString * const TimeStampKey = @"timestamp"```)

2. Write an initializer that takes a NSDictionary as a parameter and sets the timestamp, title, and text properties using the values from the dictionary.
    * note: Check to see if any of the expected values from the NSDictionary are nil, return nil if any of the values are missing
    * note: A safety check here is not required, but is the equivalent of a ```guard let``` in Swift

### Add NSUserDefaults Functionality to the EntryController

Our EntryController object is the source of truth for entries. We are now adding a layer of persistent storage, so we need to update our EntryController to load entries from NSUserDefaults on initialization, and save the entries to NSUserDefaults when they are updated.

1. Write a method called ```- (void)saveToPersistentStorage``` that will save the current entries array to NSUserDefaults
    * note: Manually loop through the entries ```NSArray``` to serialize an ```NSArray``` of plist compatible dictionary copies
    * note: Avoid 'Magic Strings' when saving to NSUserDefaults

2. Write a method called ```- (void)loadFromPersistentStorage``` that will load saved dictionary entries from NSUserDefaults and set self.entries to the results
    * note: Use the Entry ```initWithDictionary(NSDictionary *)dictionary``` while looping through the array to turn the dictionaries into Entry class objects

3. Call the ```- (void)loadFromPersistentStorage``` method when the ```EntryController``` is initialized

4. Call the ```- (void)saveToPersistentStorage``` any time that the list of entries is modified

### Black Diamonds

* Implement the NSCoding protocol on the Entry class
* Create a Unit test that verifies NSCoding methodality by converting an instance to and from NSData
* Refactor persistence to work natively with Entry objects

## Contributions

Please refer to CONTRIBUTING.md.

## Copyright

© DevMountain LLC, 2015. Unauthorized use and/or duplication of this material without express and written permission from DevMountain, LLC is strictly prohibited. Excerpts and links may be used, provided that full and clear credit is given to DevMountain with appropriate and specific direction to the original content.