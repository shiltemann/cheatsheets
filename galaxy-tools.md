# Galaxy Tools Cheat Sheet

just some examples for me to copy-paste from, definitely not complete or accurate

### Skeleton

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
    <param name="pname" type="select" label="description" help="">
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
### Outputs

**dataset:**
```xml
<data name="myoutput" format="datatype" label="${tool.name} on ${on_string}: description"/>

<!-- format from input dataset -->
<data name="myoutput" format_source="infile" label="${tool.name} on ${on_string}: description"/>

<!-- with filter -->
<data name="myoutput" format="datatype" label="${tool.name} on ${on_string}: description">
    <filter>infile or myconditional['pname'] == 'option1' or (not myboolean and myinteger > 42) or 'option1' in mymultiselect</filter>
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
    <repeat name="myrepeat">
        <param name="rparam1" value="val1"/>
        <param name="rparam2" value="val2"/>
    </repeat>
    <repeat name="myrepeat">
        <param name="rparam1" value="val3"/>
        <param name="rparam2" value="val4"/>
    </repeat>

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

detect filetype

```cheetah
#if $myfile.is_of_type('datatype'):
    <do something>
#else
    <do something else>
#end if
```

### Macros

Example macros.xml:

```xml
<macros>
    <xml name="something">
        <requirements>
            <requirement type="package" version="0.1">myrequirement</requirement>
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
```

Use token macro:

```xml
@MY_TOKEN@
```

anywhere in tool wrapper and it will be replaced by `sometext`