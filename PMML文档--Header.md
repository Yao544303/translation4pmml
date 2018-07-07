# 说明
* 基于PMML Version4.3
* 基本可以视为官方文档的一个翻译
* 如有疏漏，请联系yao544303963@gmail.com

# 头文件
包含了PMML文档的基本信息，例如模型的版权信息，模型的描述，以及生成该文件所用软件的信息（比如软件的名字和版本）。头文件中也会包含该PMML文件的生成时间。

# Schema
```
<xs:element name="Header">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element minOccurs="0" ref="Application"/>
      <xs:element minOccurs="0" maxOccurs="unbounded" ref="Annotation"/>
      <xs:element minOccurs="0" ref="Timestamp"/>
    </xs:sequence>
    <xs:attribute name="copyright" type="xs:string"/>
    <xs:attribute name="description" type="xs:string"/>
    <xs:attribute name="modelVersion" type="xs:string"/>
  </xs:complexType>
</xs:element>

<xs:element name="Application">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="name" type="xs:string" use="required"/>
    <xs:attribute name="version" type="xs:string"/>
  </xs:complexType>
</xs:element>

<xs:element name="Annotation">
  <xs:complexType mixed="true">
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>

<xs:element name="Timestamp">
  <xs:complexType mixed="true">
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```

# 说明
## Header
顶层标签，标记着header部分的开始。

**包含属性：**  
**copyright**： 
包含了该模型的版权信息。应用生成的PMML文件应该允许用户可以指定或替换该属性值。自PMML 4.1 版本起，该属性是可选的。  
**description**： 
这个属性包含了对模型的非特定描述。它需要包含程序在后续使用该模型时必要的信息，但不包括那些可以在application、annotation和data dictionary中可以精确描述的信息。这个属性只包含人类可读的信息。  
**modelVersion**： 
这个属性描述了模型的版本信息。相同或相似的模型可能被多次生成，所以这个模型对于区分这些非常重要。  
## Application
这个元素用来描述生成该模型的软件或应用。尽管生成的PMML模型是轻量级的，但是不同的机制会使基于同一个数据集产生的模型出现差异。  

**包含属性：**  
**name**：
生成该PMML文件的程序名称  
**version**：
生成该模型使用的应用的版本  

## Annotation
该元素记录文件修改历史。每条注释都是一个自由的文本。用户可以在这里记录他们自己的标记。

## Timestamp
该元素记录模型文件的创建时间


# REF
[PMML 4.3 - Header](http://dmg.org/pmml/v4-3/Header.html)