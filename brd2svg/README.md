# brd2svg

Brd2svg is a tool for converting EAGLE .brd files to Fritzing
parts.We make no guarantees about brd2svg's usefulness or
suitability, and furthermore the program is unsupported: we will
cheerfully ignore all questions, bug reports and feature requests.
We release brd2svg under the same open-source license as that of the
overall Fritzing project (GPL2/GPL3).

## How to build it

Brd2svg relies on the Qt Framework, and should work with versions 4 and 5.
**To build brd2svg, you first have to install the [Qt SDK](http://www.qt.io/download-open-source/)**, and then
easiest path for building it is to use **Qt Creator** (an IDE that comes with the SDK) to open brd2svg.pro. We have built and used brd2svg on both Mac and Windows (though it has been some time since anyone has used the Mac build). It should also work under Linux.

If you prefer the command line, you can also build it like this:

**Mac**

    brew install qt
    qmake -spec macx-g++ brd2svg.pro
    make

**GNU/Linux**

    qmake brd2svg.pro
    make

## How it works

First create a working folder. Inside that folder, create a
subfolder called 'brds'. Place all the .brd files you want to
convert into the brds folder--brd2svg will convert all the .brd
files in the folder in a single run.

Brd2svg first converts a .brd file into an xml representation (one
.xml file per .brd). In order to carry out the conversion, brd2svg
launches EAGLE which runs a script (a .ulp file). Therefore, **in
  order to run brd2svg you must have a version of [EAGLE](http://www.cadsoftusa.com/) installed** (5.0
and up). (With version 6.0, .brd files are already in an xml
format--the .ulp script was what you had to use in prior versions.
Someday we will change brd2svg to work directly with the newer .brd
xml format, so it will no longer be necessary to install EAGLE to
run brd2svg.)

The xml files are placed into an 'xml' folder which is created
inside the working folder. Once the .xml file for a given .brd has
been generated, subsequent runs of brd2svg will not create the .xml
file again. If you make a change to the original .brd file, or you
just want to clear the decks for some reason, delete the .xml file
of choice, and the next time brd2svg runs the .xml file will be
regenerated.

Next brd2svg creates a default .params file (one .params file per
.brd). These are placed into a 'params' folder inside the working
folder. The params file gives you a certain measure of control over
how the .xml files are turned into Fritzing parts (more about this
below). As with the .xml files, once a .params file is created, it
is not modified on subsequent runs of brd2svg.

An important thing to note about .params files, is that they are
created the first time you run brd2svg on a batch of .brd files. It
is only on subsequent runs of brd2svg that .params files are
actually used. Therefore, **you must run brd2svg on the same set
  of .brd files at least twice**. It turns out that using brd2svg
tends to be iterative, so in general you run brd2svg many times on
the same set of .brd files. Basically the process is: run
brd2svg--change the params files--run brd2svg--change the params
files... At some point, it becomes easier to modify the generated
.svg and .fzp files with other tools, but then if you run brd2svg
again on the same .brd files, your changes will be wiped out. So it
is a bit of a fine art to decide when to stop using brd2svg for a
given set of .brd files.

The Fritzing parts files generated by brd2svg are placed in a
'parts' folder inside the working folder. The subfolders inside the
parts folder are organized the same way as they are in the Fritzing
distribution: .fzp files are placed in parts/core and the svg files
are place into `parts/svg/core/breadboard`, `parts/svg/core/schematic`,
and `parts/svg/core/pcb`. There is an option to place parts into
`contrib` instead of `core` folders. 

**To test your new parts in
Fritzing**, you only need to copy the parts folder into your local
Fritzing user storage area into the 'parts' folder you find there
(in other words, this is a merge operation--which works nicely under
Windows and has become recently available under Mac; perhaps this is
also straightforward under Linux). The location of the Fritzing user
storage area varies by platform. Under Windows 7 you can find it at
something like `C:\Users\{your user name}\AppData\Roaming\Fritzing`.
Under Mac and Linux you can find it at `~/.config/Fritzing`.

Note that even after copying the files into the Fritzing local
storage area, the new parts won't be immediately visible in a Parts
Bin the next time you run Fritzing. Use the Parts Bin search field
to find the new parts. Alternatively, brd2svg also creates a bin
(.fzb) file. Once Fritzing is launched, use the File > Open menu
item to load the .fzp file and all the new parts you have created
should be visible in that bin. This only works if you have already
copied all parts files as described above.

## Usage

Brd2svg is accessed from the command line, there is no GUI. The
syntax is:

    brd2svg 
        -w <path to working folder> 
        -e <path to eagle executable> 
        -c <core or contrib> 
        -g 
        -p <subparts folder> 
        -s <2nd subparts folder> 
        -a <and folder>

Normally you won't use the -g option--this creates breakout-board
images for breadboard view. For the -p and -s options, you will
probably use the two subparts folders that ship with Fritzing:
{Fritzing}/parts/subparts and {Fritzing}/parts/subparts2d. The 'and'
folder is part of the source for brd2svg--it contains the .ulp
script as well as a file called all.packages.txt, and a sample
metadata file (see below for more about these two files). 

## Controlling brd2svg output

Fritzing parts consist of multiple files, one metadata file (.fzp)
and a set of .svg 

files which represent different views: schematic, pcb, breadboard,
and icon. Brd2svg generates the .fzp file plus three .svg files--one
for each of breadboard, schematic, and pcb view. The breadboard
image is reused for the icon view.

There are four ways you can influence what brd2svg writes into the
.fzp and .svg files (beyond what the original .brd file specifies):

1.  metadata.dif (one per working folder)
1.  .txt files in the descriptions folder (one per .brd file)
1.  all.packages.txt (tends to be used across projects, found in
    'and')
1.  .params files (one per .brd file)

The .params files deal with part metadata (the .fzp file) as well as
appearance (.svg files); the metadata.dif controls metadata (the
.fzp file) and board color (the breadboard .svg); the
all.packages.txt file controls breadboard .svg features; and the
description files are for part metadata. If both params.xml and
metadata.dif contain a value for the same piece of metadata for a
part (i.e. board color), metadata.dif overrides .params.

#### 1. metadata.dif

The metadata.dif file is a spreadsheet
that has been exported in [data interchange format](http://en.wikipedia.org/wiki/Data_Interchange_Format) (.dif). The metadata file must be called
"metadata.dif" and is placed into the working folder. Metadata.dif
is best used when you are dealing with a large set of parts; for
only a few parts modifying the individual .params files will be
sufficient.

The set of columns in metadata.dif is:

*   Name (this will become the part's title)
*   URL (the online documentation for this part)
*   Filename (the .brd file associated with this set of metadata)
*   Author
*   Family
*   Properties
*   Tags
*   Color

Tags are comma-separated strings. Properties are name/value pairs,
where the name is separated from the value by a colon and the pairs
are separated by semicolons. For example here are three properties:
"Voltage: 3.3V; Digital Pins: 14; Analog Pins: 6". 

Note: **'family' is required for every Fritzing part**. Generally
a single family contains a group of closely related parts which have
different properties. For example, a family of different resistors
each having a different value of resistance and tolerance (where
'resistance' and 'tolerance' are properties).

Color is the color of the board. It can be the usual html color
string, but the following presets are available:

*   blue, #147390
*   red, #c62717
*   green, #1f7a34
*   purple, #672e58
*   black, #1c1a1d
*   white, #fbfbfb

The url has an interesting feature in metadata.dif. Because SparkFun
boards are frequently converted to Fritzing parts, brd2svg will
attempt to follow the link given in the url by appending a '.json' and will save the resulting description
into the appropriate descriptions.txt file. 

Once a descriptions.txt
file is created, it is not overwritten in subsequent runs of
brd2svg.

Perhaps it is
best to look at the sample metadata.csv file in the project's 'and'
folder to get a sense of the spreadsheet. Remember to export the
spreadsheet as a .dif or brd2svg will ignore it. 

#### 2. description files

A description file is a text file containing a textual description
for a given part. It's a separate file, because it could contain HTML and doesn't fit 
well into the .dif file which has all the other metadata. 
The description file has the same basename as the .brd file it relates to, with a .txt extension.
The description can contain html, but it must be of the limited
variety that [QtRichText](http://doc.qt.io/qt-5/richtext-html-subset.html) can support.


#### 3. all.packages.txt

If there is a subpart (i.e. .svg file in one of the
    subparts folders) bearing the same name as a .brd 'package',
    brd2svg will insert that svg into the breadboard view image.
    All.packages.txt further determines how
certain subparts (aka 'packages') are converted to svg elements. For
example, the line 

    <package name='1210' ic='yes' />

means that if no subpart svg is found, brd2svg will draw a small
rectangular 'chip' on the board to represent that package. The line

    <map package='1x02' to='1X02_LOCK' />

means that the same subpart .svg will be used for the 1x02_LOCK
package as was used to draw the 1x02 package.

#### 4. .params

Params files are xml files containing the usual
part metadata: title, author, description, etc. But there is
much more, including setting the board color, xml to specify pin
placement in schematic view, and ways to 'nudge' or hide individual
packages. When brd2svg is run for the first time on a given .brd
file, it generates a .params file, and to some extent the .params
file is self-documenting--it is worth opening up one of these in a
text editor to get a sense of some of the options available. In
fact, please have a look at a generated params file before you try
to follow the rest of this section.

Here is a quick look at some of features of a .params file.

*   .brd files consist of multiple layers. Usually it is enough
      for brd2svg to convert layers 1 (top), 16 (bottom), 17 (pads),
      21 (top place), 22 (bottom place) and 20 (dimensions). You can
      optionally specify other layers to be converted (though this
      only affects the breadboard image). Note, if there is no
      dimensions layer in the .brd, Fritzing has no way to determine
      the overall size of the board. In this case, it is best to add
      your own dimensions layer to the original .brd file.

*   You can incorporate SVGs into the breadboard image. A line
      like `<include src='full path to whatever.svg' x='0in'
        y='0.2in'/>` will place the image 'whatever.svg' into
      the specified location on the breadboard .svg.

*   You can reposition, rotate or hide a package using 'nudges'.
      For example, the line `<nudge element='JP17'
        package='1X08' x='-1.1mm' y='0.2mm' angle='90' />`
      will move the JP17 element and give it an additional 90-degree
      rotation (rotations are additive, not absolute). Note that the
      element and package refer to entities found in the .xml file
      generated from a given .brd file.

*   Connectors in .fzp files are more-or-less equivalent to
      signals in .brd files. Connectors are split into five
      categories: power, ground, left, right and unused. In
      rectangular-type schematic view images, power connectors go on
      top, ground connectors go on the bottom, and the others are
      split between left and right. Unused connectors don't appear
      in Fritzing (so users are not burdened with them). The order
      of the connectors in the .params xml is the order they will
      appear along the side of the rectangle in schematic view. You
      can add &lt;space/&gt; elements to include some whitespace
      between connector pins (in schematic view only).

	It turns out most signals in a .brd file end up being unused
	in Fritzing. One rule of thumb is that signals of type 'pad'
	tend to be used and signals of type 'smd' tend to be
	unused--but this is only a rule of thumb. When a .brd file is
	first converted, brd2svg will make a guess at which category
	each signal belongs. This is usually a bad guess, and this is
	where most of the time is spent in working with params files.

*   Some boards feature prototyping areas--grids of connectors.
      You can specify this using the &lt;fake-vias&gt; element (this
      is a sibling to the &lt;connectors&gt; element). For example,
      the following xml snippet sets up the beginning of one such
      grid:

-

    <fake-vias>
        <connector signal='' id='173' package='1X3_STRIP' element='U$10' name='2' type='pad'/>
        <connector signal='' id='174' package='1X3_STRIP' element='U$10' name='3' type='pad'/>
        ..
    </fake-vias>

*   We refer to internal connections as 'buses'. To set up
      internal connections in a part, for example if there are
      multiple GND connectors, use the &lt;buses&gt; element
      (another &lt;connectors&gt; sibling). Here is another snippet:

-

    <buses>
        <bus>
            <connector signal='5V' id='160' package='1X12_STRIP' element='U$9' name='1' type='pad'/>
            <connector signal='VCC' id='126' package='1X22_STRIP' element='U$7' name='1' type='pad'/>
        </bus>
        ..
    </buses>

*   Sometimes a signal is not named very clearly, and it would be
      nice to provide a more explanatory label. If you don't want to
      change the original .brd file, you can use renaming in the
      .params file. The `<rename>` element is a child of the
      `<connectors>` element:

-

    <renames>
        <rename element='JP2' package='1X02' signal='N$223' to='+3V3' />
        <rename element='JP1' package='1X02' signal='N$223' to='+3V3' />
    </renames>


