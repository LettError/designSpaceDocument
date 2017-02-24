MutatorMath started out with its own reader and writer for designspaces. Since then the use of designspace has broadened and it would be useful to have a reader and writer that are independent of a specific system.

DesignSpaceDocument
===================

An object to read, write and edit interpolation systems for typefaces.

* the format was originally written for MutatorMath.
* the format is now also used in fontTools.varlib.
* Define sources, axes and instances.
* Not all values might be required by all applications.

A couple of differences between things that use designspaces:

* Varlib does not support anisotropic interpolations.
* MutatorMath and Superpolator will extrapolate over the boundaries of the axes. Varlib can not.
* Varlib requires much less data to define an instance than MutatorMath.
* The goals of Varlib and MutatorMath are different, so not all attributes are always needed.
* Need to expand the description of FDK use of deisgnspace files.

The DesignSpaceDocument object can read and write .designspace data. It imports the axes, sources and instances to very basic **descriptor** objects that store the data in attributes. Data is added to the document by creating such descriptor objects, filling them with data and then adding them to the document. This makes it easy to integrate this object in different contexts.

The **DesignSpaceDocument** object can be subclassed to work with different objects, as long as they have the same attributes.

```python
from designSpaceDocument import DesignSpaceDocument
doc = DesignSpaceDocument()
doc.read("some/path/to/my.designspace")
doc.axes
doc.sources
doc.instances
```

# Validation
Some validation is done when reading.

### Axes
* If the `axes` element is available in the document then all locations will check their dimensions against the defined axes. If a location uses an axis that is not defined it will be ignored.
* If there are no `axes` in the document, locations will accept all axis names, so that we can..
* Use `doc.checkAxes()` to reconstruct axes definitions based on the `source.location` values. If you save the document the axes will be there.

### Default font
* The source with the `copyInfo` flag indicates this is the default font.
* In mutatorMath the default font is selected automatically. A warning is printed if the mutatorMath default selection differs from the one set by `copyInfo`. But the `copyInfo` source will be used.
* If no source has a `copyInfo` flag, mutatorMath will be used to select one. This source gets its `copyInfo` flag set. If you save the document this flag will be set.
* Use `doc.checkDefault()` to set the default font.

# Rules
**The `rule` element is experimental.** Some ideas behind how rules could work in designspaces come from Superpolator. Such rules can maybe be used to describe some of the conditional GSUB functionality of OpenType 1.8. The definition of a rule is not that complicated. A rule has a name, and it has a number of conditions. The rule also contains a list of glyphname pairs: the glyphs that need to be substituted.

### Variable font instances
* In an variable font the substitution happens at run time: there are no changes in the font, only in the sequence of glyphnames that is rendered.
* The infrastructure to get this rule data in a variable font needs to be built.

### UFO instances
* When making instances as UFOs however, we need to swap the glyphs so that the original shape is still available. For instance, if a rule swaps `a` for `a.alt`, but a glyph that references `a` in a component would then show the new `a.alt`.
* But that can lead to unexpected results. So, if there are no rules for `adieresis` (assuming it references `a`) then that glyph **should not change appearance**. That means that when the rule swaps `a` and `a.alt` it also swaps all components that reference these glyphs so they keep their appearance.
* The swap function also needs to take care of swapping the names in kerning data.

# `SourceDescriptor` object
### Attributes
* `filename`: string. A relative path to the source file, **as it is in the document**. MutatorMath + Varlib.
* `path`: string. Absolute path to the source file, calculated from the document path and the string in the filename attr. MutatorMath + Varlib.
* `name`: string. Optional. Unique identifier name for this source, if there is one or more `instance.glyph` elements in the document. MutatorMath.
* `location`: dict. Axis values for this source. MutatorMath + Varlib
* `copyLib`: bool. Indicates if the contents of the font.lib need to be copied to the instances. MutatorMath. 
* `copyInfo` bool. Indicates if the non-interpolating font.info needs to be copied to the instances. Also indicates this source is expected to be the default font. MutatorMath + Varlib
* `copyGroups` bool. Indicates if the groups need to be copied to the instances. MutatorMath.
* `copyFeatures` bool. Indicates if the feature text needs to be copied to the instances. MutatorMath.
* `muteKerning`: bool. Indicates if the kerning data from this source needs to be muted (i.e. not be part of the calculations). MutatorMath.
* `muteInfo`: bool. Indicated if the interpolating font.info data for this source needs to be muted. MutatorMath.
* `mutedGlyphNames`: list. Glyphnames that need to be muted in the instances. MutatorMath.
* `familyName`: string. Family name of this source. Though this data can be extracted from the font, it can be efficient to have it right here. Varlib.
* `styleName`: string. Style name of this source. Though this data can be extracted from the font, it can be efficient to have it right here. Varlib.

```python
doc = DesignSpaceDocument()
s1 = SourceDescriptor()
s1.path = masterPath1
s1.name = "master.ufo1"
s1.copyLib = True
s1.copyInfo = True
s1.copyFeatures = True
s1.location = dict(weight=0)
s1.familyName = "MasterFamilyName"
s1.styleName = "MasterStyleNameOne"
s1.mutedGlyphNames.append("A")
s1.mutedGlyphNames.append("Z")
doc.addSource(s1)
```

# `InstanceDescriptor` object
* `filename`: string. Relative path to the instance file, **as it is in the document**. The file may or may not exist. MutatorMath.
* `path`: string. Absolute path to the source file, calculated from the document path and the string in the filename attr. The file may or may not exist. MutatorMath.
* `name`: string. Unique identifier name of the instance, used to identify it if it needs to be referenced from elsewhere in the document. 
* `location`: dict. Axis values for this source. MutatorMath + Varlib.
* `familyName`: string. Family name of this instance. MutatorMath + Varlib.
* `styleName`: string. Style name of this source. MutatorMath + Varlib.
* `postScriptFontName`: string. Postscript FontName for this instance. MutatorMath.
* `styleMapFamilyName`: string. StyleMap FamilyName for this instance. MutatorMath.
* `styleMapStyleName`: string. StyleMap StyleName for this instance. MutatorMath.
* `glyphs`: dict for special master definitions for glyphs. If glyphs need special masters (to record the results of executed rules for example). MutatorMath. 
* `kerning`: bool. Indicates if this instance needs its kerning calculated. MutatorMath.
* `info`: bool. Indicated if this instance needs the interpolating font.info calculated.

```python
i2 = InstanceDescriptor()
i2.path = instancePath2
i2.familyName = "InstanceFamilyName"
i2.styleName = "InstanceStyleName"
i2.name = "instance.ufo2"
# anisotropic location 
i2.location = dict(weight=500, width=(400,300))
i2.postScriptFontName = "InstancePostscriptName"
i2.styleMapFamilyName = "InstanceStyleMapFamilyName"
i2.styleMapStyleName = "InstanceStyleMapStyleName"
glyphMasters = [dict(font="master.ufo1", glyphName="BB", location=dict(width=20,weight=20)), dict(font="master.ufo2", glyphName="CC", location=dict(width=900,weight=900))]
glyphData = dict(name="arrow", unicodeValue=1234)
glyphData['masters'] = glyphMasters
glyphData['note'] = "A note about this glyph"
glyphData['instanceLocation'] = dict(width=100, weight=120)
i2.glyphs['arrow'] = glyphData
i2.glyphs['arrow2'] = dict(mute=False)
doc.addInstance(i2)
```
# `AxisDescriptor` object
* `tag`: string. Four letter tag for this axis. Some might be registered at the OpenType specification.
* `name`: string. Name of the axis as it is used in the location dicts. MutatorMath + Varlib.
* `labelnames`: dict. When defining a non-registered axis, it will be necessary to define user-facing readable names for the axis. Keyed by xml:lang code. Varlib. 
* `minimum`: number. The minimum value for this axis. MutatorMath + Varlib.
* `maximum`: number. The maximum value for this axis. MutatorMath + Varlib.
* `default`: number. The default value for this axis, i.e. when a new location is created, this is the value this axis will get. MutatorMath + Varlib.
* `map`: list of input / output values that can describe a warp of user space to designspace coordinates. If no map values are present, it is assumed it is [(minimum, minimum), (maximum, maximum)]. Varlib. 

```python
a1 = AxisDescriptor()
a1.minimum = 0
a1.maximum = 1000
a1.default = 0
a1.name = "weight"
a1.tag = "wght"
a1.labelnames[u'fa-IR'] = u"قطر"
a1.labelnames[u'en'] = u"Wéíght"
a1.map = [(0.0, 10.0), (401.0, 66.0), (1000.0, 990.0)]
```
# `RuleDescriptor` object
* `name`: string. Unique name for this rule. Will be used to reference this rule data.
* `conditions`: list of dicts with condition data.
* Each condition specifies the axis name it is active on and the values between which the condition is true.

```python
r1 = RuleDescriptor()
r1.name = "unique.rule.name"
r1.conditions.append(dict(name="aaaa", minimum=-10, maximum=10))
r1.conditions.append(dict(name="bbbb", minimum=-10, maximum=10))
```

# Subclassing descriptors

The DesignSpaceDocument can take subclassed Reader and Writer objects. This allows you to work with your own descriptors. You could subclass the descriptors. But as long as they have the basic attributes the descriptor does not need to be a subclass.

```python
class MyDocReader(BaseDocReader):
    ruleDescriptorClass = MyRuleDescriptor
    axisDescriptorClass = MyAxisDescriptor
    sourceDescriptorClass = MySourceDescriptor
    instanceDescriptorClass = MyInstanceDescriptor
    
class MyDocWriter(BaseDocWriter):
    ruleDescriptorClass = MyRuleDescriptor
    axisDescriptorClass = MyAxisDescriptor
    sourceDescriptorClass = MySourceDescriptor
    instanceDescriptorClass = MyInstanceDescriptor

myDoc = DesignSpaceDocument(KeyedDocReader, KeyedDocWriter)
```

# Document xml structure

*	The `axes` element contains one or more `axis` elements.
*	The `sources` element contains one or more `source` elements.
* 	The `instances` element contains one or more `instance` elements.

```xml
<?xml version='1.0' encoding='utf-8'?>
<designspace format="3">
	<axes>
		<!-- define axes here -->
		<axis../>
		</axes>
	<sources>
		<!-- define masters here -->
		<source../>
	</sources>
	<instances>
		<!-- define instances here -->
		<instance../>
	</instances>
</designspace>
```
# 1. `axis` element
* Define a single axis
* Child element of `axes`

### Attributes
* `name`: required, string. Name of the axis that is used in the location elements.
* `tag`: required, string, 4 letters. Some axis tags are registered in the OpenType Specification.
* `minimum`: required, number. The minimum value for this axis.
* `maximum`: required, number. The maximum value for this axis.
* `default`: required, number. The default value for this axis.

```xml
<axis name="weight" tag="wght" minimum="-1000" maximum="1000" default="0">
```

# 1.1 `labelname` element
* Defines a human readable name for UI use.
* Optional for non-registered axis names.
* Can be localised with `xml:lang`
* Child element of `axis`

### Attributes
* `xml:lang`: required, string. [XML language definition](https://www.w3.org/International/questions/qa-when-xmllang.en)

### Value
* The natural language name of this axis.

### Example
```xml
<labelname xml:lang="fa-IR">قطر</labelname>
<labelname xml:lang="en">Wéíght</labelname>
```

# 1.2 `map` element
* Defines a single node in a series of input value / output value pairs.
* Together these values transform the designspace.
* Child of `axis` element.

### Example
```xml
<map input="0.0" output="10.0" />
<map input="401.0" output="66.0" />
<map input="1000.0" output="990.0" />
```

### Example of all axis elements together:
```xml
    <axes>
        <axis default="0" maximum="1000" minimum="0" name="weight" tag="wght">
            <labelname xml:lang="fa-IR">قطر</labelname>
            <labelname xml:lang="en">Wéíght</labelname>
        </axis>
        <axis default="0" maximum="1000" minimum="0" name="width" tag="wdth">
            <map input="0.0" output="10.0" />
            <map input="401.0" output="66.0" />
            <map input="1000.0" output="990.0" />
        </axis>
    </axes>
```

# 2. `location` element
* Defines a coordinate in the design space.
* Dictionary of axisname: axisvalue
* Used in `source`, `instance` and `glyph` elements.

# 2.1`dimension` element
* Child element of `location`

### Attributes
* `name`: required, string. Name of the axis.
* `xvalue`: required, number. The value on this axis.
* `yvalue`: optional, number. Separate value for anisotropic interpolations.

### Example
```xml
<location>
  <dimension name="width" xvalue="0.000000" />
  <dimension name="weight" xvalue="0.000000" yvalue="0.003" />
</location>
```

# 3. `source` element
* Defines a single font that contributes to the designspace.
* Child element of `sources`

### Attributes
* `familyname`: optional, string. The family name of the source font. While this could be extracted from the font data itself, it can be more efficient to add it here.
* `stylename`: optional, string. The style name of the source font.
* `name`: required, string. A unique name that can be used to identify this font if it needs to be referenced elsewhere.
* `filename`: required, string. A path to the source file, relative to the root path of this document. The path can be at the same level as the document or lower.

# 3.1 `lib` element
* `<lib copy="1" />`
* Child element of `source`
* Defines if the instances can inherit the data in the lib of this source.
* MutatorMath only

# 3.2 `info` element
* `<info copy="1" />`
* Child element of `source`
* Defines if the instances can inherit the non-interpolating font info from this source.
* MutatorMath + Varlib
* This presence of this element indicates this source is to be the default font.

# 3.3 `features` element
* `<features copy="1" />`
* Defines if the instances can inherit opentype feature text from this source.
* Child element of `source`
* MutatorMath only

# 3.4 `glyph` element
* Can appear in `source` as well as in `instance` elements.
* In a `source` element this states if a glyph is to be excluded from the calculation.
* MutatorMath only

### Attributes
* `<glyph mute="1" name="A"/>`
* `mute`: optional, number, andts
* MutatorMath only

### Example
```xml
<source familyname="MasterFamilyName" filename="masters/masterTest1.ufo" name="master.ufo1" stylename="MasterStyleNameOne">
	<lib copy="1" />
   <features copy="1" />
	<info copy="1" />
   	<glyph mute="1" name="A" />
   <glyph mute="1" name="Z" />
   <location>
      <dimension name="width" xvalue="0.000000" />
      <dimension name="weight" xvalue="0.000000" />
   </location>
</source>
```
# 4. `instance` element

* Defines a single font that can be calculated with the designspace.
* Child element of `instances`
* For use in Varlib the instance element really only needs the names and the location. The `glyphs` element is not required.
* MutatorMath uses the `glyphs` element to describe how certain glyphs need different masters, mainly to describe the effects of conditional rules in Superpolator.

### Attributes
* `familyname`: required, string. The family name of the instance font. Corresponds with `font.info.familyName`
* `stylename`: required, string. The style name of the instance font. Corresponds with `font.info.styleName`
* `name`: required, string. A unique name that can be used to identify this font if it needs to be referenced elsewhere.
* `filename`: string. Required for MutatorMath. A path to the instance file, relative to the root path of this document. The path can be at the same level as the document or lower.
* `postscriptfontname`: string. Optional for MutatorMath. Corresponds with `font.info.postscriptFontName`
* `stylemapfamilyname`: string. Optional for MutatorMath. Corresponds with `styleMapFamilyName`
* `stylemapstylename `: string. Optional for MutatorMath. Corresponds with `styleMapStyleName`

### Example for varlib
```xml
<instance familyname="InstanceFamilyName" filename="instances/instanceTest2.ufo" name="instance.ufo2" postscriptfontname="InstancePostscriptName" stylemapfamilyname="InstanceStyleMapFamilyName" stylemapstylename="InstanceStyleMapStyleName" stylename="InstanceStyleName">
<location>
	<dimension name="width" xvalue="400" yvalue="300" />
   <dimension name="weight" xvalue="500" />
</location>
<kerning />
<info />
</instance>
```

# 4.1 `glyphs` element
* Container for `glyph` elements.
* Optional
* MutatorMath only.

# 4.2 `glyph` element
* Child element of `glyphs`
* May contain a `location` element.

### Attributes
* `name`: string. The name of the glyph.
* `unicode`: string. Unicode value for this glyph, in hexadecimal.

# 4.2.1 `note` element
* String. The value corresponds to glyph.note in UFO.

# 4.2.2 `masters` element
* Container for `master` elements
* These `master` elements define an alternative set of glyph masters for this glyph.

# 4.2.2.1 `master` element
* Defines a single alternative master for this glyph.

### Attributes
* `glyphname`: the name of the alternate master glyph.
* `source`: the identifier name of the source this master glyph needs to be loaded from

### Example
```xml
<instance familyname="InstanceFamilyName" filename="instances/instanceTest2.ufo" name="instance.ufo2" postscriptfontname="InstancePostscriptName" stylemapfamilyname="InstanceStyleMapFamilyName" stylemapstylename="InstanceStyleMapStyleName" stylename="InstanceStyleName">
<location>
	<dimension name="width" xvalue="400" yvalue="300" />
   <dimension name="weight" xvalue="500" />
</location>
<glyphs>
	<glyph name="arrow2" />
	<glyph name="arrow" unicode="0x4d2">
	<location>
   		<dimension name="width" xvalue="100" />
      		<dimension name="weight" xvalue="120" />
	</location>
 	<note>A note about this glyph</note>
	<masters>
		<master glyphname="BB" source="master.ufo1">
		<location>
			<dimension name="width" xvalue="20" />
			<dimension name="weight" xvalue="20" />
		</location>
		</master>
	</masters>
	</glyph>
</glyphs>
<kerning />
<info />
</instance>
```

# 5.0 `rules` element
 * Container for `rule` elements

# 5.1 `rule` element
 * Defines a named rule with a set of conditions.
* The conditional substitutions specifed in the OpenType specification can be much more elaborate than what it recorded in this element.
* So while authoring tools are welcome to use the `sub` element, they're intended as preview / example / test substitutions for the rule.

### Attributes
* `name`: required, string. A unique name that can be used to identify this rule if it needs to be referenced elsewhere.

# 5.1.1 `condition` element
* Child element of `rule`
* Between the `minimum` and `maximum` this rule is `true`.
* If `minimum` is not available, assume it is `axis.minimum`.
* If `maximum` is not available, assume it is `axis.maximum`.
* One or the other or both need to be present. 

### Attributes
* `name`: string, required. Must match one of the defined `axis` name attributes.
* `minimum`: number, required*. The low value.
* `maximum`: number, required*. The high value.

# 5.1.2 `sub` element
* Child element of `rule`.
* Defines which glyphs to replace when the rule is true.
* This element is optional. It may be useful for editors to know which glyphs can be used to preview the axis.

### Attributes
* `name`: string, required. The name of the glyph this rule looks for.
* `byname`: string, required. The name of the glyph it is replaced with.

### Example
```xml
<rules>
	<rule name="named.rule.1">
   		<condition maximum="1" minimum="0" tag="aaaa" />
		<condition maximum="3" minimum="2" tag="bbbb" />
		<sub name="dollar" byname="dollar.alt"/>
	</rule>
</rules>
```

# 6 Notes

## Paths and filenames
A designspace file needs to store many references to UFO files.

* designspace files can be part of versioning systems and appear on different computers. This means it is not possible to store absolute paths.
* So, all paths are relative to the designspace document path.
* Using relative paths allows designspace files and UFO files to be **near** each other, and that they can be **found** without enforcing one particular structure.
* The **filename** attribute in the `SourceDescriptor` and `InstanceDescriptor` classes stores the preferred relative path.
* The **path** attribute in these objects stores the absolute path. It is calculated from the document path and the relative path in the filename attribute when the object is created.
* Only the **filename** attribute is written to file. 

Right before we save we need to identify and respond to the following situations: 

In each descriptor, we have to do the right thing for the filename attribute. Before writing to file, the `documentObject.updatePaths()` method prepares the paths as follows:

**Case 1**

```
descriptor.filename == None
descriptor.path == None
```
**Action**

* write as is, descriptors will not have a filename attr. Useless, but no reason to interfere.

**Case 2**

```
descriptor.filename == "../something"
descriptor.path == None
```

**Action**

* write as is. The filename attr should not be touched.


**Case 3**

```
descriptor.filename == None
descriptor.path == "~/absolute/path/there"
```

**Action**

* calculate the relative path for filename. We're not overwriting some other value for filename, it should be fine.


**Case 4**

```
descriptor.filename == '../somewhere'
descriptor.path == "~/absolute/path/there"
```

**Action**

* There is a conflict between the given filename, and the path. The difference could have happened for any number of reasons. Assuming the values were not in conflict when the object was created, either could have changed. We can't guess.
* Assume the path attribute is more up to date. Calculate a new value for filename based on the path and the document path.

## Recommendation for editors
* If you want to explicitly set the **filename** attribute, leave the path attribute empty.
* If you want to explicitly set the **path** attribute, leave the filename attribute empty. It will be recalculated.
* Use `documentObject.updateFilenameFromPath()` to explicitly set the **filename** attributes for all instance and source descriptors.

# 7 This document

* The package is rather new and changes are to be expected.


