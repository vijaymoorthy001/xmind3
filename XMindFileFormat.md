This document describes the general specification of an XMind file.

An XMind file (workbook) is actually an archived package of several XML files and other attachments compressed using general ZIP algorithm.

# Archive Contents #
An XMind archive may contain the following components:

  * **content.xml**: (required) main data and hierarchy
  * **styles.xml**: style information
  * **meta.xml**: meta information
  * **META-INF/**: meta folder
    * **manifest.xml**: (required) the manifest of this archive
  * **attachments/**: attachment folder, used to store attached files
  * **markers/**: markers folder, used to store custom markers
  * **Thumbnails/**: thumbnail folder, used to store preview image of this workbook
    * **thumbnail.jpg**: the preview image

# XML Namespaces #
An XML file in an XMind archive may adopts two kinds of namespaces:
  * Common namespaces defined by W3C
    * **xhtml**: http://www.w3.org/1999/xhtml
    * **xlink**: http://www.w3.org/1999/xlink
    * **fo**: http://www.w3.org/1999/XSL/Format
    * **svg**: http://www.w3.org/2000/svg
  * XMind specific namespaces
    * **xmap**: urn:xmind:xmap:xmlns:content:2.0
    * **meta**: urn:xmind:xmap:xmlns:meta:2.0
    * **manifest**: urn:xmind:xmap:xmlns:manifest:1.0
    * **style**: urn:xmind:xmap:xmlns:style:2.0
    * **marker**: urn:xmind:xmap:xmlns:marker:2.0

# content.xml #
The `content.xml` contains main data and hierarchy of this workbook. The major elements are `sheet` elements and `topic` elements.

```
<xmap-content version="1.0">
    <sheet id="sheet1">
        <topic id="topic1">
            <title>Topic 1</title>
            <children>
                <topics type="attached">
                    <topic id="topic2">
                        <title>Topic 2</title>
                    </topic>
                </topics>
            </children>
        </topic>
        <relationships>
            <relationship id="relationship1" end1="topic1" end2="topic2">
            </relationship>
        </relationships>
    </sheet>
</xmap-content>
```

## xmap-content ##
The root element.
### _Attributes_ ###
  * `version`: (required) version of this `content.xml`, currently '1.0'

### _Elements_ ###
  * `sheet`: `[1,n)` the sheets of this workbook

## sheet ##
A `sheet` element represents a mind map with `topic` elements as its nodes.
### _Attributes_ ###
  * `id`: (required) the global identifier of this sheet
  * `style-id`: the global identifier of the style this sheet uses
  * `theme`: the global identifier of the theme style this sheet uses

### _Elements_ ###
  * `title`: `[0,1]` the title of this sheet
  * `topic`: `[0,1]` the root topic of this sheet
  * `relationships`: `[0,1]` the relationship container of this sheet
  * `legend`: `[0,1]` the legend of this sheet

## topic ##
A `topic` element represents a topic in a mind map. A topic may have subtopics (or none), but has only one parent (or none).

### _Attributes_ ###
  * `id`: (required) the global identifier of this topic
  * `style-id`: the global identifier of the style this topic uses
  * `xlink:href`: the hyperlink reference of this topic
  * `timestamp`: the time this topic was last modified
  * `branch`: the folding state of this topic, maybe `'folded'` or undefined
  * `structure-class`: the structure class of this topic

### _Elements_ ###
  * `title`: `[0,1]` the title of this topic
  * `image`: `[0,1]` the image of this topic
  * `notes`: `[0,1]` the notes of this topic
  * `position`: `[0,1]` the position of this topic
  * `numbering`: `[0,1]` the numbering information of this topic's subtopics
  * `children`: `[0,1]` the container of subtopics of this topic
  * `marker-refs`: `[0,1]` the container of marker references from this topic
  * `labels`: `[0,1]` the container of labels of this topic
  * `boundaries`: `[0,1]` the container of boundaries of this topic
  * `summaries`: `[0,1]` the container of summaries of this topic
  * `extensions`: `[0,1]` the container of extensions of this topic

## title ##
A `title` element represents the title of an element such as `topic`, `sheet`, `relationship`, `boundary`.

### _Attributes_ ###
  * `svg:width`: the wrap width of this title (only available in `topic`s)

### _Text Content_ ###
The text content of this element represents the content of this title.


## image ##
An `image` element represents an embedded image in a topic.

### _Attributes_ ###
  * `align`: the alignment of this image
    * 'top': image at the top of the topic
    * 'bottom': image at the bottom of the topic
    * 'left': image at the left side of the topic
    * 'right': image at the right side of the topic
  * `svg:height`: the height of this image (in pixels)
  * `svg:width`: the width of this image (in pixels)
  * `xhtml:src`: (required) the source location of this image

## notes ##
A `notes` element represents the text notes of a topic.

### _Elements_ ###
  * `plain`: `[0,1]` the plain content
  * `html`: `[0,1]` the rich content

## plain ##
A `plain` element represents the plain text part of the notes of a topic.
### _Text Content_ ###
The text content of this element represents the plain text content.

## html ##
A `html` element represents a rich text part of the notes of a topic.
### _Elements_ ###
  * `xhtml:p`: `[0,n)` the paragraphs of this text

## xhtml:p ##
A `xhtml:p` element generally follows its definition in the namespace `xhtml`, with some extra attributes and/or elements:

### _Attributes_ ###
  * `style-id`: the global identifier of the style used by this paragraph

### _Elements_ ###
  * `xhtml:span`: `[0,n)` a styled span of text
  * `xhtml:img`: `[0,n)` an image
  * `xhtml:a`: `[0,n)` a hyperlink

### _Text Content_ ###
The text content of this element represents the text content of this paragraph.

## xhtml:span ##
A `xhtml:span` element generally follows its definition in the namespace `xhtml`, with some extra attributes and/or elements:

### _Attributes_ ###
  * `style-id`: the global identifier of the style used by this span

### _Text Content_ ###
The text content of this element represents the text content of this span.

## xhtml:img ##
A `xhtml:img` element generally follows its definition in the namespace `xhtml`, with some extra attributes and/or elements:

### _Attributes_ ###
  * `xhtml:src`: the source location of this image


## xhtml:a ##
A `xhtml:a` element generally follows its definition in the namespace `xhtml`, with some extra attributes and/or elements:

### _Attributes_ ###
  * `xlink:href`: the reference location of this hyperlink

### _Elements_ ###
  * `xhtml:span`: `[0,n)` spans enclosed by this hyperlink

### _Text Content_ ###
The text content of this element represents the text content of this hyperlink.


## position ##
A `position` element represents a 2-D geographic position.

### _Attributes_ ###
  * `svg:x`: the horizontal coordinate of this position
  * `svg:y`: the vertical coordinate of this position


## numbering ##
A `numbering` element contains numbering information of a topic.

### _Attributes_ ###
  * `number-format`: (required) the format of numbers
  * `prepending-numbers`: whether to prepend numbering of the parent topic to this numbering

### _Elements_ ###
  * `prefix`: `[0,1]` the prefix content
  * `suffix`: `[0,1]` the suffix content

## prefix ##
A `prefix` element represents the prefix of the numbering text of a topic.

### _Text Content_ ###
The text content of this element represents the text content of this prefix.

## suffix ##
A `suffix` element represents the suffix of the numbering text of a topic.

### _Text Content_ ###
The text content of this element represents the text content of this suffix.

## children ##
A `children` element represents all kinds of subtopics of a topic.

### _Elements_ ###
  * `topics`: `[0,n)` the container of grouped topics that are subtopics of the parent topic

## topics ##
A `topics` element represents a group of subtopics of a parent topic. A subtopic group **MUST** specify a `type` attribute. If more than one groups have the same `type`, only the first group will be available.

### _Attributes_ ###
  * `type`: (required) the type of this group of topics
    * 'attached': topics attached to its parent topic
    * 'detached': topics not attached to its parent topic (AKA floating topic)
    * 'summary': topics attached to summaries of its parent topic

### _Elements_ ###
  * `topic`: `[0,n)` the grouped topics


## marker-refs ##
A `marker-refs` element represents the container of marker references from a topic.

### _Elements_ ###
  * `marker-ref`: `[0,n)` the references to markers

## marker-ref ##
A `marker-ref` element represents a marker reference from a topic.

### _Attributes_ ###
  * `marker-id`: the global identifier of the referenced marker

## labels ##
A `labels` element represents the container of labels of a topic.

### _Elements_ ###
  * `label`: `[0,n)` the labels

## label ##
A `label` element represents a label of a topic.

### _Text Content_ ###
The text content of this element represents the content of the label

## boundaries ##
A `boundaries` element represents the container of boundaries of a topic.

### _Elements_ ###
  * `boundary`: `[0,n)` the boundaries of this topic

## boundary ##
A `boundary` element represents a boundary of a topic.'

### _Attributes_ ###
  * `id`: (required) the global identifier of this boundary
  * `style-id`: the global identifier of the style used by this boundary
  * `range`: (required) the range of this boundary
    * '(_start_, _end_)': the boundary starts from the _start_-th subtopic to the _end_-th subtopic of the parent topic
    * 'master': the boundary covers the whole parent topic

### _Elements_ ###
  * `title`: `[0,1]` title of this boundary

## summaries ##
A `summaries` element represents the container of summaries of a topic.

### _Elements_ ###
  * `summary`: `[0,n)` summaries of this topic

## summary ##
A `summary` element represents a summary of a topic.

### _Attributes_ ###
  * `id`: (required) the global identifier of this summary
  * `style-id`: the global identifier of the style used by this summary
  * `range`: (required) '(_start_, _end_)' this summary starts from the _start_-th subtopic to the _end_-th subtopic of the parent topic
  * `topic-id`: the global identifier of the summary topic

## extensions ##
An `extensions` element represents the container of topic extensions.

### _Elements_ ###
  * `extension`: `[0,n)` extensions of the topic

## extension ##
An `extension` element represents a topic extension. A topic extension is used to store additional information of this topic, generally written/read by specific extension provider.

### _Attributes_ ###
  * `provider`: (required) the provider of this topic extension
    * if multiple topic extensions have the same provider, only the first one will be available.

### _Elements_ ###
  * `content`: `[0,1]` the content of this extension
  * `resource-refs`: `[0,1]` the container of resource references used by this extension

## content ##
A `content` element represents the content of a topic extension. This element is not restricted and is available to store any information provided by the extension provider.

### _Attributes_ ###
(not restricted)

### _Elements_ ###
(not restricted)

## resource-refs ##
A `resource-refs` element represents the container of resource references from a topic extension.

### _Elements_ ###
  * `resource-ref`: `[0,n)` the resource references

## resource-ref ##
A `resource-ref` element represents a resource reference from a topic extension.

### _Attributes_ ###
  * `resource-id`: (required) the global identifier of this resource
  * `type`: (required) the type of this resource (currently only 'file-entry' is supported)
    * 'file-entry'

## relationships ##
A `relationships` element represents the container of relationships in a sheet.

### _Elements_ ###
  * `relationship`: `[0,n)` relationships of this sheet

## relationship ##
A `relationship` element represents a relationship between two elements such as `topic` and/or `boundary`.

### _Attributes_ ###
  * `id`: (required) the global identifier of this relationship
  * `style-id`: the global identifier of the style used by this relationship
  * `end1`: (required) the global identifier of one end of this relationship
  * `end2`: (required) the global identifier of the other end of this relationship

### _Elements_ ###
  * `title`: `[0,1]` the title of this relationship
  * `control-points`: `[0,1]` the container of control points of this relationship

## control-points ##
A `control-points` element represents the container of control points of a relationship.

### _Elements_ ###
  * `control-point`: `[0,2]` control points of this relationship

## control-point ##
A `control-point` element represents a control point of a relationship.

### _Attributes_ ###
  * `index`: (required) the index of this control point
    * currently only two control points is supported, so `index` should be either `0` or `1`

### _Elements_ ###
  * `position`: `[0,1]` the relative position of this control point


## legend ##
A `legend` element represents the map legend of a sheet, used to display marker information of this sheet.

### _Attributes_ ###
  * `visibility`: the visibility of this legend
    * 'visible'
    * 'hidden'

### _Elements_ ###
  * `position`: `[0,1]` the position of this legend on the sheet
  * `marker-descriptions`: `[0,1]` the container of marker descriptions of this legend

## marker-descriptions ##
A `marker-descriptions` element represents the container of marker description items of a legend.

### _Elements_ ###
  * `marker-description`: `[0,n)` the marker description of the legend

## marker-desciption ##
A `marker-description` element represents a marker description item in a legend.

### _Attributes_ ###
  * `marker-id`: (required) the global identifier of the marker being described
  * `description`: (required) the description of this marker in this sheet


# styles.xml #
The `styles.xml` contains style information of this workbook. The major elements are `style` elements.

## xmap-styles ##
The root element.

### _Attributes_ ###
  * `version`: (required) the version of this `styles.xml`, currently `1.0`

## Elements ##
  * `styles`: normal styles
  * `master-styles`: master styles
  * `automatic-styles`: automatic styles

## styles ##
A `styles` element represents the normal style group of a style sheet.

### _Elements_ ###
  * `style`: `[0,n)` styles in this group


## master-styles ##
A `master-styles` element represents the master style group of a style sheet.

### _Elements_ ###
  * `style`: `[0,n)` styles in this group


## automatic-styles ##
A `automatic-styles` element represents the automatic style group of a style sheet.

### _Elements_ ###
  * `style`: `[0,n)` styles in this group


## style ##
A `style` element represents a style.

### _Attributes_ ###
  * `id`: (required) the global identifier of this style
  * `type`: (required) the type of this style
    * 'topic'
    * 'map'
    * 'boundary'
    * 'relationship'
    * 'summary'
    * 'theme'
  * `name`: the name of this style


### _Elements_ ###
  * _xxx_`-properties`: `[0,n)` _xxx_ must be the the `type` attribute of this style element

## xxx-properties ##
This kind of elements represent sets of properties of a style.

### _Attributes_ ###
  * `angle`
  * `arrow-begin-class`
  * `arrow-end-class`
  * `background`
  * `fo:background-color`
  * `fo:color`
  * `svg:fill`
  * `fo:text-decoration`
  * `fo:font-family`
  * `fo:font-size`
  * `fo:font-style`
  * `fo:font-weight`
  * `svg:height`
  * `line-class`
  * `line-color`
  * `line-corner`
  * `line-pattern`
  * `line-tapered`
  * `line-width`
  * `fo:margin-bottom`
  * `fo:margin-left`
  * `fo:margin-right`
  * `fo:margin-top`
  * `multi-line-colors`
  * `svg:opacity`
  * `shape-class`
  * `shape-corner`
  * `spacing-major`
  * `spacing-minor`
  * `fo:text-align`
  * `fo:text-bullet`
  * `svg:width`
  * `svg:x`
  * `svg:y`

### _Elements_ ###
  * `default-style`: `[0,n)` styles that describe specific style applied for different style family (**only available within `theme-properties` element**)

## default-style ##
A `default-style` element represents a default style to be used for elements of different style families.

### _Attributes_ ###
  * `style-family`: (required) the family name of the described style
    * 'centralTopic'
    * 'mainTopic'
    * 'subTopic'
    * 'floatingTopic'
    * 'relationship'
    * 'boundary'
    * 'summary'
    * 'map'
  * `style-id`: (required) the global identifier of the referenced style


# markerSheet.xml #
The `markerSheet.xml` contains marker information of this workbook, used to describe custom markers embedded in this workbook. System default markers will never be embedded.

## marker-sheet ##
The root element.

### _Elements_ ###
  * `marker-group`: `[0,n)` groups of markers

## marker-group ##
A `marker-group` element represents a group of markers.

### _Attributes_ ###
  * `id`: the global identifier of this marker group
  * `name`: the name of this marker group
  * `singleton`: whether this marker group is singleton
    * No more than one marker from a singleton group can be added on a topic

### _Elements_ ###
  * `marker`: `[0,n)` markers in this group

## marker ##
A `marker` element represents a marker.

### _Attributes_ ###
  * `id`: the global identifier of this marker
  * `name`: the name of this marker
  * `resource`: the location of the resource used to display this marker (e.g. an image file)


# META-INF/manifest.xml #
The `manifest.xml` contains file entry information of this workbook archive.
```
<manifest xmlns="urn:xmind:xmap:xmlns:manifest:1.0">
    <file-entry full-path="attachments/" media-type=""/>
    <file-entry full-path="attachments/2luj06vdfu84ar5pe5bq5ujol8.png" media-type="image/png"/>
    <file-entry full-path="content.xml" media-type="text/xml"/>
    <file-entry full-path="META-INF/" media-type=""/>
    <file-entry full-path="META-INF/manifest.xml" media-type="text/xml"/>
    <file-entry full-path="meta.xml" media-type="text/xml"/>
    <file-entry full-path="styles.xml" media-type=""/>
    <file-entry full-path="Thumbnails/" media-type=""/>
    <file-entry full-path="Thumbnails/thumbnail.jpg" media-type="image/jpeg"/>
</manifest>
```

## manifest ##
The root element.

### _Elements_ ###
  * `file-entry`: `[0,n)` the file entries of this workbook

## file-entry ##
A `file-entry` element represents a file entry in this workbook

### _Attributes_ ###
  * `full-path`: the full path of this file entry within the archive
  * `media-type`: the media type of this file entry

### _Elements_ ###
  * `encryption-data`: `[0,1]` the encryption data of this file entry (if encrypted)

## encryption-data ##
(To be done)

# meta.xml #
(To be done)