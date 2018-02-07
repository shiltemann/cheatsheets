# Galaxy Tools Cheat Sheet

just some examples for me to copy-paste from, definitely not complete or accurate

### Shed yaml

Create a `.shed.yml` file for automatic upload to toolshed using Planemo:

```yaml
name: toolname
owner: owner
categories:
  - ToolShedCategory1
  - ToolShedCategory2
description: Tool description
long_description: |
    Longer tool description
    newlines are okay
remote_repository_url: https://github.com/project/repository
homepage_url: http://www.example.org
type: unrestricted
```

To make a suite of tools, add the following:

```yaml
auto_tool_repositories:
  name_template: "{{ tool_id }}"
  description_template: "Wrapper for application X: {{ tool_name }}."
suite:
  name: "suite_name"
  description: "A suite of tools that does a cool thing"
  long_description: |
    Longer tool description
    newlines are okay
```

### Tool Skeleton

```xml
<tool id="tool_id" name="tool_name" version="0.1" profile="16.07">
    <description>tool description</description>
    <macros>
        <import>macros.xml</import>
    </macros>
    <requirements>
        <requirement type="package" version="0.1">name</requirement>
    </requirements>
    <version_command>myprog --version</version_command>
    <command detect_errors="aggressive"><![CDATA[
your command goes here
    ]]></command>
    <inputs>
    </inputs>
    <outputs>
    </outputs>
    <tests>
        <test>
        </test>
    </tests>
    <help><![CDATA[
help text goes here
    ]]></help>
    <citations>
        <citation type="doi">42.4242/42</citation>
    </citations>
</tool>
```

### Params

**dataset:**
```xml
<param name="myfile" type="data" format="datatype" optional="false" argument="--clarg" label="description" help=""/>
```

**dataset collection:**  
```xml
<param name="input" type="data_collection" collection_type="pair|list" label="Input Collection"/>
```

**text:**
```xml
<param name="mytext" type="text" value="default" size="100x5" label="description" help=""/>
```

**numbers:**
```xml
<param name="myinteger" type="integer" value="10" min="0" max="100" label="description" help=""/>
<param name="myfloat" type="float" value="0.1" min="0.0" max="1.0" label="description" help=""/>
```

**boolean:**
```xml
<param name="myboolean" type="boolean" truevalue="--param" falsevalue="" checked="true" label="description" help=""/>
```

**select:**
```xml
<param name="myselect" type="select" multiple="false" label="description" help="">
    <option value="option1">description1</option>
    <option value="option2" selected="true">description2</option>
</param>

<param name="myselect" type="select" label="description">
    <options from_data_table="mydatatable"/>
</param>
```

when `multiple=true` value is comma-separated string `option1,option2,option42`

**conditionals:**
```xml
<conditional name="myconditional">
    <param name="conditional_pname" type="select" label="description" help="">
        <option value="option1">description1</option>
        <option value="option2" selected="true">description2</option>
    </param>
    <when value="option1">
        <param name="pname2" type="data" format="datatype" optional="false" label="description" help=""/>
    </when>
    <when value="option2">
        <param name="pname3" type="data" format="datatype" optional="false" label="description" help=""/>
    </when>
</conditional>
```

in command: `$myconditional.pname` and `$myconditional.pname2`

**repeats:**

```xml
<repeat name="myrepeat" min="1" title="mytitle">
    <param name="rparam1" type="ptype" .. />
    <param name="rparam2" type="ptype" .. />
</repeat>
```

in cheetah

```cheetah
#for $i $myrepeat
    echo $i.rparam1
#end if

#for $i, $s in enumerate( $myrepeat )
    echo "repeat number $s: $i.rparam1"
#end if
```

**sections:**

```xml
<section name="mysection" title="Advanced settings">
    <param name="sectionparam1" type="ptype" .. />
    <param name="sectionparam2" type="ptype" .. />
    ...
</section>
```

in command: `$mysection.sectionparam1`

**validators:**

[documentation](https://docs.galaxyproject.org/en/latest/dev/schema.html#id31)

```xml
<param name="myparam" type="text">
    <validator type="empty_field" />
    <validator type="length" min="42" message="length of your input must be at least 42" />
    <validator type="regex" message="please match my regex">\S+</validator>
</param>

<param name="myparam" type="data">
    <validator type="unspecified_build" />
    <validator type="dataset_metadata_in_data_table" table_name="my_indexes" metadata_name="dbkey" metadata_column="dbkey" message="no indices currently available for the specified build." />
</param>

<param name="myparam" type="integer|float">
    <validator type="in_range" min="0" max="100"/>
</param>

<param name="myparam" type="select">
    <validator type="no_options" message="A built-in reference genome is not available for the build associated with the selected input file"/>
</param>
```

### Outputs

**dataset:**
```xml
<data name="myoutput" format="datatype" label="${tool.name} on ${on_string}: description"/>

<!-- format from input dataset -->
<data name="myoutput" format_source="infile" label="${tool.name} on ${on_string}: description"/>

<!-- with filter -->
<data name="myoutput" format="datatype" label="${tool.name} on ${on_string}: description">
    <filter>infile
            or myconditional['conditional_pname'] == 'option1'
            or (not myboolean and myinteger > 42)
            or 'option1' in mymultiselect
            or myparam.ext == 'ftype'
    </filter>
</data>
```

**collection:**
```xml
<collection name="mycollection" type="list" label="${tool.name} on ${on_string}: description">
    <discover_datasets pattern=".*?\.(?P&lt;designation&gt;.*)\.suffix" directory="subdir" format="datatype"/>
</collection>
```

### Tests

```xml
<test><!-- test description -->
    <param name="myfile" value="filename.in" ftype="datatype"/>
    <param name="myparam" value="myvalue"/>
    <param name="conditional_pname" value="option1"/>
    <param name="sectionparam1" value="42"/>
    <repeat name="myrepeat">
        <param name="rparam1" value="val1"/>
        <param name="rparam2" value="val2"/>
    </repeat>
    <repeat name="myrepeat">
        <param name="rparam1" value="val3"/>
        <param name="rparam2" value="val4"/>
    </repeat>
    <param name="my_input_collection">
        <collection type="list">
            <element name="sample1" ftype="ftype" value="file1.ext" />
            <element name="sample2" ftype="ftype" value="file2.ext" />
        </collection>
    </param>

    <output name="outputname" file="filename.out" ftype="datatype" lines_diff="x"/>
    <output name="outputname" md5="ed659c0af6b78f710f53f45ea99b9bb" ftype="datatype"/>
    <output name="outputname" ftype="datatype">
        <assert_contents>
            <has_text text="text"/>
            <not_has_text text="notext"/>
            <has_text_matching expression="myregex"/>
            <has_line_matching expression="^myregex$"/>
        </assert_contents>
    </output>
    <output_collection name="mycollection" count="25">
        <element name="elementname" file="filename.out" ftype="datatype"/>
    </output_collection>
</test>
```

### Command

access script from install directory:

```cheetah
${__tool_directory__}/myscript.sh
python ${__tool_directory__}/myscript.py
Rscript ${__tool_directory__}/myscript.R
...
```

detect filetype (will also match subtypes, `$myfile.extension` will not):

```cheetah
#if $myfile.is_of_type('datatype'):
    <do something>
#else
    <do something else>
#end if
```

set number of processors:

```cheetah
\${GALAXY_SLOTS:-4}
```

(if `GALAXY_SLOTS` not set, this uses value 4)

### Macros

Example `macros.xml`:

```xml
<macros>
    <xml name="something">
        <requirements>
            <requirement type="package" version="0.1">myrequirement</requirement>
            <yield/> <!-- if you want to be able to add more requirements in tool xml itself -->
        </requirements>
    </xml>
    <token name="@MY_TOKEN@">sometext</token>
</macros>
```

In the tool wrapper

Import macro file:

```xml
<macros>
    <import>macros.xml</import>
</macros>
```

Use xml macro:

```xml
<expand macro="something"/>
<expand macro="something_with_a_yield">
    <requirement type="package" version="42">extra-requirement</requirement>
</expand>
```

Use token macro:

```xml
@MY_TOKEN@
```

anywhere in tool wrapper and it will be replaced by `sometext`
