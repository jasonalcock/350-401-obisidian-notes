---
aliases: 
publish: 
---
%%
date:: [[2024-09-15]]
parent:: 
%%
# [[XML]]

- XML stands for eXtensible Markup Language.
- XML was designed to store and transport data.
- XML was designed to be both human- and machine-readable.

![[Pasted image 20240915101915.png]]

- XML has a root element
- All elements can have text content (Harry Potter) and attributes (category="cooking")
- An element can contain:
	- text
	- attributes
	- other elements
	- or a mix of the above

```xml
<?xml version="1.0" encoding="UTF-8**"**?>  
<bookstore>  
  <book category="cooking">  
    <title lang="en">Everyday Italian</title>  
    <author>Giada De Laurentiis</author>  
    <year>2005</year>  
    <price>30.00</price>  
  </book>  
  <book category="children">  
    <title lang="en">Harry Potter</title>  
    <author>J K. Rowling</author>  
    <year>2005</year>  
    <price>29.99</price>  
  </book>  
  <book category="web">  
    <title lang="en">Learning XML</title>  
    <author>Erik T. Ray</author>  
    <year>2003</year>  
    <price>39.95</price>  
  </book>  
</bookstore>
```

- This is called the prologue and is optional `<?xml version="1.0" encoding="UTF-8**"**?>` 

#### Namespaces

## Name Conflicts

In XML, element names are defined by the developer. This often results in a conflict when trying to mix XML documents from different XML applications.
- Name conflicts in XML can easily be avoided using a name prefix : then element name e.g `<x:element name>`.
```xml
<h:table>  
  <h:tr>  
    <h:td>Apples</h:td>  
    <h:td>Bananas</h:td>  
  </h:tr>  
</h:table>  
  
<f:table>  
  <f:name>African Coffee Table</f:name>  
  <f:width>80</f:width>  
  <f:length>120</f:length>  
</f:table>
```
- When using prefixes in XML, a **namespace** for the prefix must be defined
```xml
<root>  
  
<h:table xmlns:h="http://www.w3.org/TR/html4/">  
  <h:tr>  
    <h:td>Apples</h:td>  
    <h:td>Bananas</h:td>  
  </h:tr>  
</h:table>  
  
<f:table xmlns:f="https://www.w3schools.com/furniture">  
  <f:name>African Coffee Table</f:name>  
  <f:width>80</f:width>  
  <f:length>120</f:length>  
</f:table>  
  
</root>
```
- Namespaces can also be declared in the XML root element or in the element itself
- The namespace URI is not used by the parser to look up information.
- The purpose of using an URI is to give the namespace a unique name.
- However, companies often use the namespace as a pointer to a web page containing namespace information
- A **Uniform Resource Identifier** (URI) is a string of characters which identifies an Internet Resource.
- The most common URI is the **Uniform Resource Locator** (URL) which identifies an Internet domain address. Another, not so common type of URI is the **Uniform Resource Name** (URN).
- Defining a default namespace for an element saves us from using prefixes in all the child elements. It has the following syntax:
	- xmlns="_namespaceURI_"
```xml
<table xmlns="https://www.w3schools.com/furniture">  
  <name>African Coffee Table</name>  
  <width>80</width>  
  <length>120</length>  
</table>
```

#### References
- [XML Tree](https://www.w3schools.com/xml/xml_tree.asp)