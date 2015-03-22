This document describes how to use the XMind API to programmatically generate and update mind maps using xmind.

# Introduction #
The Xmind object model is built around a number of core interfaces that provide nearly all the functionality needed to create, manipulate and save mind maps.  These interfaces are defined in the core plugin org.xmind.core:
  * **`org.xmind.core.IWorkbook`**  - represents an entire mind map, which may include multiple sheets and style information, as well as images , attachments and other files.
  * **`org.xmind.core.ISheet`** – represents a single sheet within a mind map.  Each sheet has a central topic.
  * **`org.xmind.core.ITopic`** – represents a topic within a sheet on a mind map.  A topic has a single parent topic (unless it is the central topic for a sheet) and many have any number of children
In addition to these main interfaces there are several more used for specific purposes.  Some of these are:
  * **`org.xmind.core.IRelationship`** – represents a relationship between any two topics
  * **`org.xmind.core.IControlPoint`** – represents a point on the relationship used to set the curve
  * **`org.xmind.core.IBoundary`** – a grouping of adjacent topics
  * **`org.xmind.core.ISummary`** – similar to a boundary, but with a topic node attached to it
  * **`org.xmind.core.IImage`** – an image attached to a topic
  * **`org.xmind.core.INumbering`** – a numbering schema attached to a topic and its children
  * **`org.xmind.core.ITopicExtension`** – allow for custom extensions to be added to a topic
  * **`org.xmind.core.IWorkBookBuilder`** – used to create or open workbook files
  * **`org.xmind.core.style.IStyleSheet`** – container of all the styles used in a workbook
  * **`org.xmind.core.style.IStyle`** – used to add style information to various objects, including topics, sheets and relationships
  * **`org.xmind.core.event.ICoreEventSource`** – event listener sources
  * **`org.xmind.core.event.ICoreEventSupport`** – mechanism to fire event listeners
  * **`org.xmind.core.IManifest`** – control the xmind document manifest for adding attachments
  * **`org.xmind.core.marker.IMarkerSheet`** - contains all marker groups
  * **`org.xmind.core.marker.IMarkerGroup`** - collection of related markers.
  * **`org.xmind.core.marker.IMarker`** - small icon added to topic
  * **`org.xmind.core.marker.IMarkerRef`** - reference to a marker
  * **`org.xmind.core.marker.IMarkerResource`** - resource to work with icon image for marker
Finally, there are also some base classes with static methods and fields that are needed:
  * **`org.xmind.core.Core`** – common event definitions, as well as static methods to instantiate many of the interfaces
  * **`org.xmind.core.util.Point`** – used to set the coordinates for detached topics
  * **`org.xmind.ui.style.Styles`** – holds all the style definitions for both properties and values

# Workbooks #
## Obtaining a workbook ##
The first step is to get the `IWorkbook` for the mind map you are working on.  The `IWorkbookBuilder` interface has methods to get new workbooks – either creating one or opening one that already exists.
```
String Workbook = “C:`*`path`*`to`*`newWorkbook.xmind“;
String oldWorkbook = “C:`*`path`*`to`*`oldWorkbook.xmind“;

IWorkbookBuilder builder = Core.getWorkbookBuilder();
IWorkbook Workbook = builder.createWorkbook(newWorkbookPath);
IWorkbook oldWorkbook = builder.loadFromPath(oldWworkbookPath);
```
## Saving a workbook ##
It doesn't do much good if you can't save the work you do with a mind map.  The `IWorkbook` has several methods to facility this
```
String getFile();  // return the name of the current default file
void setFile(String file);  // set the name of the current default file

void save();    // save the workbook to the current default file
void save(String file);  // save the workbook to the specified file name
void save(OutputStream output);  // save the workbook to the specified OutputStream
void save(IOutputTarget target);  // save the workbook to the target
```
In addition to the workbook, the developer may need to be aware of the temporary storage.  This is where all the data is written
while the workbook is open.  If there are not attachments then the developer will rarely need to be aware of these, however when
manipulating attachments it is crucial.

The key method here is `saveTemp()` which writes all data to the temporary storage.  Attachments that have been added to the manifest
are not visible until the workbook has been saved to temporary storage, so it is good practice to call `saveTemp()` after creating a
new attachment.
```
    void saveTemp();

    void setTempStorage(IStorage storage);
    IStorage getTempStorage();
    void setTempLocation(String tempLocation);
    String getTempLocation();
```

# Working with Topics #
## Creating and attaching Topics ##
Once you have access to the `IWorkbook` you can then start adding topics.  Every `IWorkbook` has a default sheet, and every sheet has a root topic.  This is where you start for adding additional topics.

```
ISheet defSheet = workbook.getPrimarySheet();
ITopic rootTopic = defSheet.getRootTopic();
rootTopic.setTitleText(“Root Topic”);
```

To create a new topic, use the workbook again:
```
ITopic topic1 = workbook.createTopic();
ITopic topic2 = workbook.createTopic();
ITopic topic3 = workbook.createTopic();
```

In order to be used, a topic must be added to a map.  This is done by adding a topic as a child of another topic.  There are two types of children.  Attached children are direct subtopics of their parent, and will always stay with the parent until detached.  Attached topics inherit the structure of the parent. In addition, Xmind will auto-place all attached topics when it draws the diagram.  Detached children, on the other hand, are manually placed on the diagram and will not move (unless the overlap is disabled, in which case they may be 'bumped' by other topics).  Detached topics do not inherit anything from the parent.
To control whether a topic is attached or detached, set the appropriate flag when adding the child:
```
rootTopic.add(topic1, ITopic.ATTACHED);
```
Children of a topic are kept in the order added (and displayed in that order).  However you can insert a child in a specific position instead of at the end by specifying a (zero based) index in the add() method.  The following line inserts topic2 as the first topic in the mind map:
```
rootTopic.add(topic2, 0, ITopic.ATTACHED);
```
When adding a detached child topic, it is important to set the location of the topic.  Any detached topic which does not have a location set will default to the center of the diagram, where it will be bumped downward by the central topic.  If you add several detached topics without specifying their location, then you will get a list of them below the root topic, which is usually not what you want.
Detached child topics are also ordered, however this order is only significant is no position is specified for the topic.  Additionally, attached and detached topics are kept in separate lists, so be aware of this when using the index.
To set the location of a detached topic, simply create a `Point` object with the appropriate X/ Y values (relative to the center of the screen (which is also where the root topic is located)).
```
Point detPoint = new Point(500, -400);
topic3.setPosition(detPoint);
rootTopic.add(topic3, ITopic.DETACHED);
```
## Topic Settings ##
There are some basic methods on the topic that you will use to specify information on it.  Most of these are simple and obvious.
### Title Text ###
Title text is a key attribute of a topic; without it you will generally see nothing but an empty box when displaying the map.  In addition to setting the title, you can also set the width.  When displaying titles that are shorter than the width
specified for them, Xmind will wrap the title text to the next line.  In addition, you can use the styles (see below) to set the alignment for the text.
```
topic1.setTitleText(“first subtopic”);
topic1.setTitleWidth(8);

topic2.setTitleText(“second subtopic”);
topic3.setTitleText(“third (detached) topic”);
```
### Folding Topics ###
For large maps, it is often inconvenient to have all topics on the map displayed all the time.  There is a functionality to collapse (or fold) the subtopics of a topic.  When the topic is folded, then subtopics only appear as a **+** icon.
Clicking this in the UI will display the subtopics again.  This is set in the API via the `setFolded(boolean folded)` method.
```
topic3.setFolded(true); // hides the subtopics

topic2.setFolded(false); // display the subtopics (default)
```
### Hyperlinks ###
Adding a hyperlink to a topic puts an icon within the topic that, within the UI, can be clicked to follow the hyperlink.  There are three types of hyperlinks that can be set on a topic:
  1. URL
  1. topic
  1. attachment (file)

URL's are the simplest type of hyperlink - simply specify the string value of the URL
```
topic2.setHyperlink(“http://code.google.com/p/xmind3/w/list”);  // URL
```
Topics are also not difficult to link.  Only topics within the current map can be linked however.  The format of a topic link is `xmind:#topicID` - where topicID is the internal ID of the topic.  Be aware that topic id's are not permanent, and are regenerated each time a map is loaded.
```
topic1.seHyperlink("xmind:#" + topic2.getId()); // topic
```
Attachments are slightly more difficult to manage.  In order for attachments to work, the file itself has to be put into the xmind document.  Details on creating attachments are below, however the
link for them is setup as follows
```
topic3.setHyperlink("xap:attachments/filename"); // attachment
```
### Structure ###
Xmind has several types of structures available for displaying maps.  The method to set these is simple - however it takes a bit of digging within the API to get the proper values to use.  The method `setStructureClass(String classname)` on `ITopic` takes
a string that is the full classname of the class that implements the structure.
```
topic2.setStructureClass("org.xmind.ui.fishbone.leftHeaded");
```
The following table lists the various structures availble, and the classes for each.  Always check the API for the version you are using to validate these.
| **Structure Type** | **Structure class name**|
|:-------------------|:|
| Map (default) | "" - empty string |
| Left Headed Fishbone | `org.xmind.ui.fishbone.leftHeaded` |
| Right Headed Fishbone | `org.xmind.ui.fishbone.rightHeaded` |
| Left Logic Diagram | `org.xmind.ui.logic.left` |
| Right Logic Diagram | `org.xmind.ui.logic.right` |
| Downward Organization Chart | `org.xmind.ui.org-chart.down` |
| Upward Organization Chart | `org.xmind.ui.org-chart.up` |
| Spreadsheet | `org.xmind.ui.spreadsheet` |
| Left Tree | `org.xmind.ui.tree.left` |
| Right Tree | `org.xmind.ui.tree.right` |
| Floating | `org.xmind.ui.map.floating` |

### Labels ###
Labels are displayed under a topic, and can provide for additional information.  A topic can have multiple labels, each is displayed on a separate line.
To set labels, simply pass a `Collection<String>` to the `setLabel()` method.
```
Collection<String> labels = new ArrayList<String>();
labels.add("label1");
labels.add("label2");
topic2.setLabels(labels);
```
### Notes ###
Notes display on topics as icons, and provide the capability to rollover the icon to display, or click on them to bring up a box and edit them.  Because of these
capabilities, they are a little more difficult to set than setting a label.

To create a new note, first create an `INotesContent`, which there are two types - `IPlainNotesContent` and `IHtmlNotesContent`.  For this discussion we will set a plain notes content,
setting html content is a bit more involved, but similar.  Once the content object is created, then the appropriate content is set.

An `INotes` object is obtained from the topic.  If the topic had a note set on it, then it is returned.  Otherwise a new object is created and attached to the topic.  This then is assigned the content that has been built.
```
IPlainNotesContent plainContent = (IPlainNotesContent) workbook.createNotesContent(INotes.PLAIN);
plainContent.setTextContent("This is the note");
INotes notes = topic1.getNotes();
notes.setContent(INotes.PLAIN, plainContent);
```
### Numbering ###
Numbering allows you to use xmind like an outline and automatically number all the descendents of a node.  There are actually four parameters to setting numbering,
  * format - string describing number format
  * prefix - static string appended before the number
  * suffix - static string appended after the number
  * prepends - is the parent topic number prepended to this topic's number
To set the numbering properties, an `INumbering` object is obtained from the topic.  If one does not exist, then it is created automatically for you.  The format expects
specific values to indicate the available formats
```
INumbering num = topic1.getNumbering();
num.setFormat("org.xmind.numbering.arabic");
num.setPrefix("Num: ");
num.setSuffix("x");
num.setPrependsParentNumbers(true);
```
| **Number format name** | **Number format string value** |
|:-----------------------|:-------------------------------|
| None | org.xmind.numbering.none |
| Arabic (1, 2, 3, ...) | org.xmind.numbering.arabic |
| Roman (I, II, III, ...) | org.xmind.numbering.roman |
| a, b, c, ... | org.xmind.numbering.lowercase |
| A, B, C, ... | org.xmind.numbering.uppercase |

## Setting the style (or making topics look pretty) ##

There are many more settings available for customizing topics, including colors, fonts, shapes and structure.  All of these are also available through the API, though they take a little more work to access since they are not set directly on the objects themselves.

In fact, many objects within a mind map have style settings, and all of them are set the same way as they are set for a topic.

In order to change the stylistic elements of a topic, a `IStyle` object is needed.  These are all kept in the `IWorkbook` in an attached `IStyleSheet`. For a topic you find its current `IStyle` in the `IStylesheet` for the `IWorkbook`, and if it does not exist then you create one and add it to the `IStyleShee`t:
```
IStyleSheet styleSheet = workbook.getStyleSheet();
IStyle style = styleSheet.findStyle(topic1.getStyleId());
if (style == null) {
  style = styleSheet.createStyle(IStyle.TOPIC);
  styleSheet.addStyle(style, IStyleSheet.NORMAL_STYLES);
}
```
Once you have set the properties on the `IStyle`, you then assign it back to the topic.
```
topic1.setStyleId(style.getId());
```
One thing to be aware of however – if you are using listeners (see below) on your mind map (including viewers and editors), then changing the style does **not**, by itself, fire the listeners.  In fact you can change everything about an existing style, but if the style id on the topic does not change then no listeners are fired.

There are two ways around this.  One simple method is to clear the styleId on the topic and then set it back to the value – however this does have the effect of firing the listeners twice (once to clear the style, and once to set it again).  A better, but less obvious method is to fire the listeners manually when the style changes:
```
if (topic1 instanceof ICoreEventSource) {
    ICoreEventSource source = (ICoreEventSource) topic1;
    ICoreEventSupport ces = source.getCoreEventSupport();
    ces.dispatchValueChange(source, Core.Style, "", style.getId());
}
```
Working with styles is one of the few areas that is not contained within the `org.xmind.core.jar` archive.  It is located instead in `org.xmind.ui.mindmap.jar`.  This makes sense as styles are much more about how the mind map looks, not the actual structure itself.

When setting styles, the class `org.xmind.ui.style.Styles` is the critical component.  This class has no methods, only containing static values to define both style names and values.  An `IStyle` is _effectively_ a hashmap of name/value pairs that describe how an object is different than the default, as defined by `Styles`.

Once you have an `IStyle` object for a topic, then changing the style is done simply by setting the appropriate property name with the appropriate value:
```
style.setProperty(Styles.FillColor, “0x000000”);  // fill color black
style.setProperty(Styles.Textcolor, “0xffffff”);  // text color white
```
The following table is a list of the appropriate style names and values as defined in `Styles`.    This is not meant to be an exhaustive list, but as a quick reference.  Always check the API for the version you are using.

| **Styles Name** | **Styles values** |
|:----------------|:------------------|
| ShapeClass | DEFAULT |
|  | TOPIC\_SHAPE\_ROUNDEDRECT |
|  | TOPIC\_SHAPE\_RECT |
|  | TOPIC\_SHAPE\_ELLIPSE |
|  | TOPIC\_SHAPE\_UNDERLINE |
|  | TOPIC\_SHAPE\_DIAMOND |
|  | TOPIC\_SHAPE\_CALLOUT\_ELLIPSE |
|  | TOPIC\_SHAPE\_CALLOUT\_ROUNDEDRECT |
|  | TOPIC\_SHAPE\_NO\_BORDER |
| LineClass | DEF\_BRANCH\_DECORATION |
|  | BRANCH\_CONN\_STRAIGHT |
|  | BRANCH\_CONN\_CURVE |
|  | BRANCH\_CONN\_ARROWED\_CURVE |
|  | BRANCH\_CONN\_ROUNDEDELBOW |
|  | BRANCH\_CONN\_ELBOW |
|  | BRANCH\_CONN\_NONE |
| FillColor | DEF\_TOPIC\_FILL\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| LineColor | DEF\_TOPIC\_LINE\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| TextColor | DEF\_TEXT\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| LinePattern | DEFAULT |
|  | LINE\_PATTERN\_SOLID |
|  | LINE\_PATTERN\_DASH |
|  | LINE\_PATTERN\_DOT |
|  | LINE\_PATTERN\_DASH\_DOT |
|  | LINE\_PATTERN\_DASH\_DOT\_DOT |
| LineWidth | DEFAULT |
|  | 1pt`*`|
|  | 2pt`*`|
|  | 3pt`*`|
|  | 4pt`*`|
|  | 5pt`*`|
| FontStyle | DEFAULT |
|  | FONT\_STYLE\_ITALIC |
| FontWeight | DEFAULT |
|  | FONT\_WEIGHT\_BOLD |
| FontSize | DEFAULT |
|  | 8pt`*`|
|  | 9pt`*`|
|  | 10pt`*`|
|  | 11pt`*`|
|  | 12pt`*`|
|  | 14pt`*`|
|  | 16pt`*`|
|  | 18pt`*`|
|  | 20pt`*`|
|  | 22pt`*`|
|  | 24pt`*`|
|  | 36pt`*`|
|  | 48pt`*`|
|  | 56pt`*`|
|  | 64pt`*`|
|  | 72pt`*`|
| TextDecoration | DEFAULT |
|  | TEXT\_DECORATION\_UNDERLINE |
|  | TEXT\_DECORATION\_LINE\_THROUGH |
|  | TEXT\_UNDERLINE\_AND\_LINE\_THROUGH |
| TextAlign | DEFAULT |
|  | ALIGN\_CENTER |
|  | ALIGN\_LEFT |
|  | ALIGN\_RIGHT |

`*` These values are strings that are not defined in Styles

`*``*` Colors are specified via the RGB color scheme – which is a 6 digit hexadecimal number, effectively allowing for 16 million colors, though it is much more common to use the 216 standard web safe colors.

## Attachments ##
### Files ###
Xmind stores its files in an open document format, which is a zipped file containing xml files as well as other files necessary for the document.  Included among these
are attachments, which may be images, documents or other files.  When using the UI it is simple enough to drag and drop and attachment onto a topic, or to use the menu
to add an attachment.  Using the API however, requires you to store the file within the document, then to make a hyperlink to it in the appropriate topic.  In addition,
if this is an image that is being attached, then there are additional parametes available to describe the location and size of the attachment.

For files that are **not** images on a topic (even though these files may be images, they are not displayed in the topic), the manifest is gotten from the workbook, and then
the file is added to the manifest, generating a file entry.  From the file entry you can then get the pathname to use in the topic hyperlink.
```
IManifest manifest = workbook.getManifest();
```
There are two ways to create a new attachment - from an input stream or from a string containing the path to the file.  The following two examples are equivalent.
Open a stream and create the attachment from it:
```
String directory = "C:\files\";
String filename = "attachment.txt";
String filePath = directory + filename;

FileInputStream fis = new FileInputStream(filePath);
IFileEntry entry = manifest.createAttachmentFromStream(fis, fileName);
```
Create an attachment directly from a file path:
```
String directory = "C:\files\";
String filename = "attachment.txt";
String filePath = directory + filename;

IFileEntry entry = manifest.createAttachmentFromFilePath(filePath);
```
It is important to note that an attachment is a **copy** of the original file - the original file is not changed, and changes to it **will not** be reflected within the mind map.
Once the attachment has been created, it is then linked to the topic via hyperlink with the `xap:` protocol:
```
topic2.setHyperlink("xap:" + entry.getPath()); // attachment
```
### Images ###
Images can be added to a mind map in two ways.  One is as simple attachments - in which case they are treated as any other file attachment (see above).  The second
is to embed the image within the topic itself, so that it displays on the map.  This takes some additional work.

The image file (.jpg, .gif, .png) must first be added to the document like any other attachment.  Then instead of adding a hyperlink to the file, an `IImage` object
is obtained from the topic (either the existing image, or if one does not exist, a new object is returned), and then the image file is assigned to this, as well as other attributes.
```
IImage image = topic3.getImage();
image.setSource("xap:" + entry.getPath());
image.setHeight(80); // pixels
image.setWidth(80); // pixels
image.setAlignment("left"); // one of "top", "right", "bottom", "left"
```
## Markers ##
Markers are icons that can be added to topics.  Markers are contained in groups, and only a single marker per group may be added to a topic, however there is no limit
to the number of groups of markers that a single topic can contain.

Adding a marker to a topic is simply done by passing the markerId into the topic.  The tricky part is to get the markerId.  There are a large set of standard markers available
within the API, however custom markers can also be generated.

In order work with markers, an `IMarkerSheet` is obtained from the workbook.  The sheet contains the marker groups, which contain the markers.  New groups can be created,
and new markers can be added within groups
```
IMarkerSheet markerSheet = workbook.getMarkerSheet();

IMarkerGroup markerGroup = markerSheet.findMarkerGroup("Flags");
List<IMarkerGroup> groupList = markerSheet.getMarkerGroups();

IMarker marker = markerGroup.getMarker("flag-red");
List<IMarker> markerList = markerGroup.getMarkers();
```
One tricky point when working with the existing markers is that the marker name and marker id are _not_ the same.  Once a marker is obtained, then it is simple
to add it to the topic, or remove it, or to see if it already exists.  In addition you can also get all the markers on a topic
```
topic3.addMarker(marker.getId());

topic2.removeMarker(marker.getId());

boolean has Marker = topic1.hasMarker(marker.getId());

Set<IMarkerRef> markerSet = topic1.getMarkerRefs();
```
Custom marker can be added as well.  One of the simplest ways is to import a directory of icons (which are simply small image files, _not_ icon files) into a
marker sheet
```
String iconDirectory = "C:\myicons";
IMarkerSheet sheet = workbook.getMarkerSheet();
sheet.importFrom(iconDirectory);
```
The other method is to add individual markers.  The marker is created with an id, then the name is added.  An IMarkerResource is then obtained from the marker
and the image is written to it.
```
public IMarker createMarker(String markerName, String iconPath, IWorkbook workbook, IMarkerGroup group) {
  IMarker marker = null;
  IMarkerSheet ims = workbook.getMarkerSheet();
  InputStream is = new FileInputStream(iconFile);
  if (is != null) {
    marker = ims.createMarker(iconFile.getName());
    marker.setName(markerName);
    IMarkerResource resource = marker.getResource();
    OutputStream os = resource.getOutputStream();
    org.xmind.core.util.FileUtils.transfer(is, os, false);
  }
  group.addMarker(marker);
  return marker;
}
```
## Moving Topics ##
Topics can be moved quite easily using the API.  Either to another parent, or their position within in the list of children for the parent.  Either way it is
done by removing the topic from the parent, and then adding it to the new parent (which can be the same one), setting the index value.
```
root.remove(topic2);
topic1.add(topic2, 2, ITopic.ATTACHED);
```
## Boundaries ##
Boundaries are a way of grouping consecutive topics.  The boundary is actually attached to the parent topic, with the start and end indices of the children
contained within the boundary listed.  First the `IBoundary` is created and then it is added to the topic.  Removing and getting all the boundaries is also simple.
```
IBoundary boundary = workbook.createBoundary();
boundary.setStartIndex(0);
boundary.setEndIndex(2);
topic2.addBoundary(boundary);

// removing a boundary
topic2.removeBoundary(boundary);

// getting all the boundaries for a topic
Set<IBoundary> boundarySet = topic2.getBoundaries();
```
Boundaries do not actually do anything in the abstract map, however they do display an outline around the sub-topics they contain.  As an item that is displayed,
boundaries can also have styles associated with them, just like topics.  Changing the style of a boundary is nearly identical to changing the style of a topic,
and boundaries have the same methods to get and set the style id as topics.
```
IStyleSheet styleSheet = workbook.getStyleSheet();
IStyle style = styleSheet.findStyle(boundary.getStyleId());
if (style == null) {
  style = styleSheet.createStyle(IStyle.BOUNDARY);
  styleSheet.addStyle(style, IStyleSheet.NORMAL_STYLES);
}
```

| **Styles Name** | **Styles values** |
|:----------------|:------------------|
| ShapeClass | DEFAULT |
|  | BOUNDARY\_SHAPE\_ROUNDEDRECT |
|  | BOUNDARY\_SHAPE\_RECT |
|  | BOUNDARY\_SHAPE\_SCALLOPS |
|  | BOUNDARY\_SHAPE\_WAVES |
|  | BOUNDARY\_SHAPE\_TENSION |
| FillColor | DEF\_TOPIC\_FILL\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| Opacity | 0.0 - 1.00 - percentage value `*``*``*` |
| LineWidth | DEFAULT |
|  | 1pt`*`|
|  | 2pt`*`|
|  | 3pt`*`|
|  | 4pt`*`|
|  | 5pt`*`|
| LineColor | DEF\_TOPIC\_LINE\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| TextColor | DEF\_TEXT\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| FontStyle | DEFAULT |
|  | FONT\_STYLE\_ITALIC |
| FontWeight | DEFAULT |
|  | FONT\_WEIGHT\_BOLD |
| FontSize | DEFAULT |
|  | 8pt`*`|
|  | 9pt`*`|
|  | 10pt`*`|
|  | 11pt`*`|
|  | 12pt`*`|
|  | 14pt`*`|
|  | 16pt`*`|
|  | 18pt`*`|
|  | 20pt`*`|
|  | 22pt`*`|
|  | 24pt`*`|
|  | 36pt`*`|
|  | 48pt`*`|
|  | 56pt`*`|
|  | 64pt`*`|
|  | 72pt`*`|
| TextDecoration | DEFAULT |
|  | TEXT\_DECORATION\_UNDERLINE |
|  | TEXT\_DECORATION\_LINE\_THROUGH |
|  | TEXT\_UNDERLINE\_AND\_LINE\_THROUGH |
| TextAlign | DEFAULT |
|  | ALIGN\_CENTER |
|  | ALIGN\_LEFT |
|  | ALIGN\_RIGHT |

`*``*``*` Opacity is expressed as a two digit percentage - i.e. 24% is 0.24
## Summaries ##
A summary is similar to a boundary.  Like a boundary it is added to a parent specifying a continuous set of subtopics.  However a summary also generates a new topic of type
ITopic.SUMMARY.  Adding, removing and listing summaries are all done the same way as boundaries.
```
ISummary summary = workbook.createSummary();
summary.setStartIndex(0);
summary.setEndIndex(2);
topic2.addSummary(summary);

// removing a summary
topic2.removeSummary(summary);

// getting all the summaries for a topic
Set<ISummary> summarySet = topic2.getSummaries();

ITopic sumTopic = summary.getTopic();
```
Summaries also have styles, and the topic for a summary has it's own style (see the section on setting styles on topics).  One restriction is the topic in a summary cannot be removed,
it is an essential part of the summary.

```
IStyleSheet styleSheet = workbook.getStyleSheet();
IStyle style = styleSheet.findStyle(summary.getStyleId());
if (style == null) {
  style = styleSheet.createStyle(IStyle.SUMMARY);
  styleSheet.addStyle(style, IStyleSheet.NORMAL_STYLES);
}
```

| **Styles Name** | **Styles values** |
|:----------------|:------------------|
| ShapeClass | DEFAULT |
|  | SUMMARY\_SHAPE\_ANGLE |
|  | SUMMARY\_SHAPE\_ROUND |
|  | SUMMARY\_SHAPE\_SQUARE |
|  | SUMMARY\_SHAPE\_CURLY |
| LineWidth | DEFAULT |
|  | 1pt`*`|
|  | 2pt`*`|
|  | 3pt`*`|
|  | 4pt`*`|
|  | 5pt`*`|
| LineColor | DEF\_TOPIC\_LINE\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| TextColor | DEF\_TEXT\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| FontStyle | DEFAULT |
|  | FONT\_STYLE\_ITALIC |
| FontWeight | DEFAULT |
|  | FONT\_WEIGHT\_BOLD |
| FontSize | DEFAULT |
|  | 8pt`*`|
|  | 9pt`*`|
|  | 10pt`*`|
|  | 11pt`*`|
|  | 12pt`*`|
|  | 14pt`*`|
|  | 16pt`*`|
|  | 18pt`*`|
|  | 20pt`*`|
|  | 22pt`*`|
|  | 24pt`*`|
|  | 36pt`*`|
|  | 48pt`*`|
|  | 56pt`*`|
|  | 64pt`*`|
|  | 72pt`*`|
| TextDecoration | DEFAULT |
|  | TEXT\_DECORATION\_UNDERLINE |
|  | TEXT\_DECORATION\_LINE\_THROUGH |
|  | TEXT\_UNDERLINE\_AND\_LINE\_THROUGH |
| TextAlign | DEFAULT |
|  | ALIGN\_CENTER |
|  | ALIGN\_LEFT |
|  | ALIGN\_RIGHT |

## Extending Topics ##
An extremely powerful capability within the Xmind API that is not available from the UI is the capability to add extensions to topics.  The topic extension mechanism
allows the developer to add and store additional data within an xmind topic that is persisted within the mind map document.  Any XML serializable object can be stored
within an extension, and multiple extensions can be added to a topic.  The possibilities there are endless.

The following example shows how to store a String within a topic and retrieve it
```
private static final String EXTENSION = "sample";

public void setStringExtension(ITopic topic, String selectionString) {
  ITopicExtension extension = topic.getExtension(EXTENSION);
  if(extension == null) {
    extension = topic.createExtension(EXTENSION);
  }
  ITopicExtensionElement content = extension.getContent();
  if (content != null) {
    content.setTextContent(selectionString);
  }
}

public String getStringExtension(ITopic topic) {
  String selection = null;
  ITopicExtension extension = topic.getExtension(EXTENSION);
  if(extension != null) {
    ITopicExtensionElement content = extension.getContent();
    if (content != null) {
      selection = content.getTextContent();
    }
  }
  return selection;
}

public void removeStringExtension(ITopid topic) {
  topic.deleteExtension(EXTENSION);
}
```
## Miscellaneous topic methods ##
It wouldn't be very good if we were able to set all these attributes on topics but were not able to identify what had been set on one.  There are a wide variety
of utility methods to do this.  The usage of these methods should be obvious
```
topic2.getType(); // returns one of ITopic.ROOT, ITopic.ATTACHED, ITopic.DETACHED, ITopic.SUMMARY

boolean isAttached = topic3.isAttached();

boolean isRoot = topic1.isRoot();

boolean isFolded = topic2.isFolded();

int index = topic3.getIndex(); // returns the index of this topic within the list of its parents children

String hyperlink = topic2.getHyperlink();

String structure = topic1.getStructureClass();

String title = topic2.getTitleText();

int titleWidth = topic2.getTitleWidth();

long time = topic3.getModifiedTime();
```
### Getting children ###
A mind map is a tree, beginning at the root node.  Often there are times when you need to walk the tree to get all the sub-topics, either of the root (and thus the entire map)
or a topic within the map.
```
boolean hasChildren = topic1.hasChildren(ITopic.ATTACHED); // or ITopic.DETACHED or ITopic.SUMMARY

List<ITopic> allChildren = root.getAllChildren();
List<ITopic> attachedChildren = topic2.getChildren(ITopic.ATTACHED); // type is either ITopic.ATTACHED or ITopic.DETACHED or ITopic.SUMMARY
```

### searching for topics ###
One drawback with working with the Xmind API is that the ids for the objects are not persisted - they are recreated each time the map is loaded.  This results in a slight
dilema however, as often the only way to access an object without searching the entire map programmatically is by the object id.

For example, `IWorkbook` has a method `public ITopic findTopic(String topicId);`.  This will return the topic with the given id.  How do you get the id for a topic?  With
the method `String getId()' on the topic itself.  There is not, however, a method to set the id, as this is generated each time the map is loaded.  This leaves the developer
in a catch-22 situation - they can get a topic if they have it's id, but they can only get the id from the topic.

None of the information on a topic, aside from its generated id, is actually unique.  Developers may find it useful to keep a `java.util.Map` of topics to provide fast
access to individual topics, building it when the mind map is initially loaded and updating it as topics are added or deleted.  The map can be indexed on whatever relevant
information the developer needs to identify topics, be it title, color or even the value of an extension.

# Sheets #
Topics are not the only objects that the developer will need to work with.  It is quite common for a mind map to break into multiple sheets.  Sheets are no more difficult
to work with than topics, though they do not have nearly as many options.

To work with sheets, you either need to get an existing sheet, or create one.  Sheets, like subtopics, are kept in an ordered list, so it is possible to insert
a sheet into a specific place in the list
```
ISheet primarySheet = workbook.getPrimarySheet();  // get primary sheet

List<ISheet> sheetList = workbook.getSheets();  // get all sheets

ISheet sheet = workbook.createSheet();  // create a new sheet

workbook.addSheet(sheet);  // add a sheet to the end of the list

ISheet sheet2 = workbook.createSheet();
workbook.addSheet(sheet, 0);  // add sheet as the first sheet in the list

workbook.movesheet(2, 0);  // move the sheet in position 2 to the front of the list
```
Of course, sheets have styles as well
```
IStyleSheet styleSheet = workbook.getStyleSheet();
IStyle style = styleSheet.findStyle(sheet.getStyleId());
if (style == null) {
  style = styleSheet.createStyle(IStyle.SHEET);
  styleSheet.addStyle(style, IStyleSheet.NORMAL_STYLES);
}
```

| **Styles Name** | **Styles values** |
|:----------------|:------------------|
| FillColor | DEF\_SHEET\_FILL\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| Opacity | 0.0 - 1.00 - percentage value `*``*``*` |
| LineTapered | tapered |
| background | `*``*``*``*` |

`*``*``*``*` Sheet backgrounds are a special case of a style.  The value here is actually an attachment file (like topic attachments).
The opacity style only applies to background.
```
String backgroundFilePath = "C:\files\background.jpg";
IManifest manifest = workbook.getManifest();

IFileEntry entry = manifest.createAttachmentFromFilePath(backgroundFilePath);
style.setProperty(STYLE.background, "xap:" + entry.getPath());
```
## Relations ##
In a mind map, the major relationship between topics is hierarchical, with a given parent topic having one or more subtopics etc.  However, there is a
capability to model relationships between any two topics.  These are called, appropriately, relationships.  A relationship is attached to a sheet, and is defined
between two topics.
```
IRelationship relation = workbook.createRelationship();
relation.setEnd1Id(topic1.getId());
relation.setEnd2Id(topic2.getId());
relation.setTitleText("Relationship Description");
sheet.addRelationship(relation);
```
And, of course, relationships have styles
```
IStyleSheet styleSheet = workbook.getStyleSheet();
IStyle style = styleSheet.findStyle(relation.getStyleId());
if (style == null) {
  style = styleSheet.createStyle(IStyle.RELATIONSHIP);
  styleSheet.addStyle(style, IStyleSheet.NORMAL_STYLES);
}
```

| **Styles Name** | **Styles values** |
|:----------------|:------------------|
| ShapeClass | DEFAULT |
|  | REL\_SHAPE\_CURVED |
|  | REL\_SHAPE\_ANGLE |
|  | REL\_SHAPE\_STRAIGHT |
| LinePattern | DEFAULT |
|  | LINE\_PATTERN\_SOLID |
|  | LINE\_PATTERN\_DASH |
|  | LINE\_PATTERN\_DOT |
|  | LINE\_PATTERN\_DASH\_DOT |
|  | LINE\_PATTERN\_DASH\_DOT\_DOT |
| LineWidth | DEFAULT |
|  | 1pt`*`|
|  | 2pt`*`|
|  | 3pt`*`|
|  | 4pt`*`|
|  | 5pt`*`|
| LineColor | DEF\_TOPIC\_LINE\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| ArrowBeginClass | ARROW\_SHAPE\_NORMAL |
|  | ARROW\_SHAPE\_NORMAL |
|  | ARROW\_SHAPE\_SPEARHEAD |
|  | ARROW\_SHAPE\_DOT |
|  | ARROW\_SHAPE\_TRIANGLE |
|  | ARROW\_SHAPE\_SQUARE |
|  | ARROW\_SHAPE\_DIAMOND |
|  | ARROW\_SHAPE\_HERRINGBONE |
|  | ARROW\_SHAPE\_NONE |
| ArrowEndClass | ARROW\_SHAPE\_NORMAL |
|  | ARROW\_SHAPE\_NORMAL |
|  | ARROW\_SHAPE\_SPEARHEAD |
|  | ARROW\_SHAPE\_DOT |
|  | ARROW\_SHAPE\_TRIANGLE |
|  | ARROW\_SHAPE\_SQUARE |
|  | ARROW\_SHAPE\_DIAMOND |
|  | ARROW\_SHAPE\_HERRINGBONE |
|  | ARROW\_SHAPE\_NONE |
| TextColor | DEF\_TEXT\_COLOR |
|  | NONE |
|  | Color RGB Value in hex – ex 0xffffff for Black `*``*` |
| FontStyle | DEFAULT |
|  | FONT\_STYLE\_ITALIC |
| FontWeight | DEFAULT |
|  | FONT\_WEIGHT\_BOLD |
| FontSize | DEFAULT |
|  | 8pt`*`|
|  | 9pt`*`|
|  | 10pt`*`|
|  | 11pt`*`|
|  | 12pt`*`|
|  | 14pt`*`|
|  | 16pt`*`|
|  | 18pt`*`|
|  | 20pt`*`|
|  | 22pt`*`|
|  | 24pt`*`|
|  | 36pt`*`|
|  | 48pt`*`|
|  | 56pt`*`|
|  | 64pt`*`|
|  | 72pt`*`|
| TextDecoration | DEFAULT |
|  | TEXT\_DECORATION\_UNDERLINE |
|  | TEXT\_DECORATION\_LINE\_THROUGH |
|  | TEXT\_UNDERLINE\_AND\_LINE\_THROUGH |
| TextAlign | DEFAULT |
|  | ALIGN\_CENTER |
|  | ALIGN\_LEFT |
|  | ALIGN\_RIGHT |

One of the interesting points with relationships is they each have two control points, which are used to control the shape of the relationship.
By default the control points lie along the line between the two topics, however they can be moved, curving the line (depending on the shape class being
used).  Each relationship has two control points, access by index.  The control point nearest the first topic is index 0, the second topic control point is
index 1.

```
Point cPoint0 = new Point(500, -400);
IControlPoint cp0 = relation.getControlPoint(0);
cp0.setPoistion(cPoint0);

Point cPoint1 = new Point(200, -100);
IControlPoint cp1 = relation.getControlPoint(1);
cp1.setPosition(cPoint1);
```
# Adding Listeners #
One of the most powerful capabilities of Xmind is the ability to listen to events for virtually anything you can do with a map.  This allows
a developer to write code triggered off of nearly anything a user can do with a map without the need to touch any of the base Xmind code.

Listeners are added via the `ICoreEventSupport` interface.  Unfortunately Xmind does not currently have a 'clean' method of obtaining this
like it does for other objects.
```
ICoreEventSupport ces - (ICoreEventSupport) workbook.getAdapter(ICoreEventSupport.class);
```
Once you have the `ICoreEventSupport`, you need to register a listener.  A listener is simple a class that implements `ICoreEventListener`:
```
class MyCoreEventListener implements ICoreEventListener {
  public void handleCoreEvent(CoreEvent event) {
    // do something
  }
}
```
Then simply register your listener for the event you wish to listen to
```
  MyCoreEventListener mcel = new MyCoreEventListener();
  ces.registerGlobalListener(Core.TopicAdd, mcel); // listen for the topic add event
```
Unless you choose to implement a separate listener class for each type of event, you will need to determine the event type of each event.  The `CoreEvent` class
has several methods to get the information from the event.  Each type of event, however, uses different parameters.

```
  public void handleCoreEvent(CoreEvent event) {
    String type = event.getType();
    ICoreEventSource ces = event.getEventSource();
    int index = evnet.getIndex();
    Object oldValue = event.getOldValue();
    Object newValue = event.etNewValue();
    Object data = event.getData();
    Object object = event.getTarget();
```
The event source is the object which generated the event - for example if it is an event about a topic, then the event source is an ITopic.   However if it is
and TopicAdd event then the source is the parent topic, while the target is the topic that is added.

Some of the more useful events to listen to are listed in the table below, along with the types of objects available in the event.  Note - this is not an exhaustive list of event - see the API for org.xmind.core.CORE
for all events.
| **Event type** | **Event Description** | **Event Source** | **New Value** | **Target** |
|:---------------|:----------------------|:-----------------|:--------------|:-----------|
| Core.BoundaryAdd | add a boundary | ITopic |  | IBoundary |
| Core.BoundaryRemove | remove a boundary | ITopic |  | IBoundary |
| Core.EndIndex | set the end index on a boundary | IBoundary |  |  |
| Core.EndIndex | set the end index on a summary | ISummary |  |  |
| Core.ImageAlignment | top or bottom | IImage |  |  |
| Core.ImageHeight | height of image | IImage |  |  |
| Core.ImageSource | image | IImage |  |  |
| Core.ImageWidth | width of image | IImage |  |  |
| Core.Labels | add labels | ITopic |  |  |
| Core.MarkerRefAdd | add a defined marker | ITopic |  | String (marker ID) |
| Core.MarkerRemove | remove a defined marker | ITopic |  | String (marker ID) |
| Core.NumberFormat | type of outline number | INumbering |  |  |
| Core.NumberingPrefix | prefix to number | INumbering |  |  |
| Core.NumberingSuffix | suffix to number | INumbering |  |  |
| Core.NumberPrepending | add the parent number | INumbering |  |  |
| Core.Position | set the position on a control point | IControlPoint | Point |  |
| Core.Position | set the position on a topic | ITopic | Point |  |
| Core.RelationshipAdd | add a relationship | ISheet |  | IRelationship |
| Core.RelationshipEnd1 | change the source of a relationship | IRelationship |  |  |
| Core.RelationshipEnd2 | change the target of a relationship | IRelationship |  |  |
| Core.RelationshipRemove | remove a relationship | ISheet |  | IRelationship |
| Core.SheetAdd | add a sheet |  |  | ISheet |
| Core.SheetMove | move a sheet |  |  | ISheet |
| Core.SheetRemove | remove a sheet |  |  | ISheet |
| Core.StartIndex | set the start index on a boundary | IBoundary |  |  |
| Core.StartIndex | set the start index on a summary | ISummary |  |  |
| Core.StructureClass | change the structure class | ITopic | String |  |
| Core.Style | set the style id on a boundary | IBoundary |  |  |
| Core.Style | set the style id on a relationship | IRelationship |  |  |
| Core.Style | set the style id on a sheet | ISheet |  |  |
| Core.Style | set the style id on a summary | ISummary |  |  |
| Core.Style | set the style id on a topic | ITopic |  |  |
| Core.SummaryAdd | add a summary | ITopic |  | ISummary |
| Core.SummaryRemove | remove a summary | ITopic |  | ISummary |
| Core.TitleText | set title text on a boundary | IBoundary | String |  |
| Core.TitleText | set title text on a relationship | IRelationship | String |  |
| Core.TitleText | set title text on a sheet | ISheet | String |  |
| Core.TitleText | set title text on a topic | ITopic | String |  |
| Core.TitleWidth | set the width of the title on a topic | ITopic | Integer |  |
| Core.TopicAdd | add topic | ITopic (parent) |  | ITopic (new) |
| Core.TopicFolded | hide children | ITopic |  |  |
| Core.TopicHyperlink | set a hyperlink / attachment | ITopic |  |  |
| Core.TopicNotes | notes | ITopic |  |  |
| Core.TopicRemove | remove topic | ITopic (parent) |  | ITopic |


# Conclusion #
This should help you to begin to create, manipulate and save mind maps via the Xmind API.  It is meant as a starting point for developers,
not as an exhaustive listing of all possible methods and API calls.