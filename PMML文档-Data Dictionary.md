# 说明
* 基于PMML Version4.3
* 基本可以视为官方文档的一个翻译
* 如有疏漏，请联系yao544303963@gmail.com

# 数据字典
数据字典包含了数据挖掘模型中使用的字段定义。指定了字段类型和字段值的取值范围。
这些定义是独立的，无关于使用了何种数据集训练了何种模型。
一个数据字典，可以被多个模型、统计值以及其他相关的操作共享。

# Schema
```
<xs:element name="DataDictionary">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element ref="DataField" maxOccurs="unbounded"/>
      <xs:element ref="Taxonomy" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="numberOfFields" type="xs:nonNegativeInteger"/>
  </xs:complexType>
</xs:element>

<xs:element name="DataField">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:sequence>
        <xs:element ref="Interval" minOccurs="0" maxOccurs="unbounded"/>
        <xs:element ref="Value" minOccurs="0" maxOccurs="unbounded"/>
      </xs:sequence>
    </xs:sequence>
    <xs:attribute name="name" type="FIELD-NAME" use="required"/>
    <xs:attribute name="displayName" type="xs:string"/>
    <xs:attribute name="optype" type="OPTYPE" use="required"/>
    <xs:attribute name="dataType" type="DATATYPE" use="required"/>
    <xs:attribute name="taxonomy" type="xs:string"/>
    <xs:attribute name="isCyclic" default="0">
      <xs:simpleType>
        <xs:restriction base="xs:string">
          <xs:enumeration value="0"/>
          <xs:enumeration value="1"/>
        </xs:restriction>
      </xs:simpleType>
    </xs:attribute>
  </xs:complexType>
</xs:element>

<xs:simpleType name="OPTYPE">      
  <xs:restriction base="xs:string">
    <xs:enumeration value="categorical"/>
    <xs:enumeration value="ordinal"/>
    <xs:enumeration value="continuous"/>
  </xs:restriction>
</xs:simpleType>

<xs:simpleType name="DATATYPE">      
  <xs:restriction base="xs:string">
    <xs:enumeration value="string"/>
    <xs:enumeration value="integer"/>
    <xs:enumeration value="float"/>
    <xs:enumeration value="double"/>
    <xs:enumeration value="boolean"/>
    <xs:enumeration value="date"/>
    <xs:enumeration value="time"/>
    <xs:enumeration value="dateTime"/>
    <xs:enumeration value="dateDaysSince[0]"/>
    <xs:enumeration value="dateDaysSince[1960]"/>
    <xs:enumeration value="dateDaysSince[1970]"/>
    <xs:enumeration value="dateDaysSince[1980]"/>
    <xs:enumeration value="timeSeconds"/>
    <xs:enumeration value="dateTimeSecondsSince[0]"/>
    <xs:enumeration value="dateTimeSecondsSince[1960]"/>
    <xs:enumeration value="dateTimeSecondsSince[1970]"/>
    <xs:enumeration value="dateTimeSecondsSince[1980]"/>
  </xs:restriction>
</xs:simpleType>
```

# 说明
## DataDictionary
该元素定义了数据字典的顶层。可以包含三种类型的子元素： Extension、DataField、Taxonomy  

**包含属性：**

**numberOfFields** 
该属性值是数据字典中定义的字段的数量，可以用来进行前后一致性的校验（字段数量的校验）

## DataField
该元素定义了字段。 
在整个数据字典中，字段名称必须唯一。也有一些特殊情况，在整个PMML文档中，名称以必须唯一。

**包含属性：**  
**displayName**：
应用程序使用该名称来关联该字段。在XML文档内部，只有**name**的值是必须的，如果没有给出**displayName**，那其默认值就是**name**的值。  
举个例子：  
某个字段的**name="SCTAGE" displayName="Customer age"**。应用程序在调用PMML 消费者时，会在接口处使用Customer age来请求输入数据。一旦消费者接受了参数，并且和miningFields匹配上了，那么**displayName**就没有什么实际意义了，在内部处理中，都是使用**name**。

**optype**：
字段的类型，即定义在这些字段值上的操作类型，主要有三类。
* categorical 类别型 只能进行相等或不相等的运算。（即，是这个分类，或者不是这个分类）
* ordinal 定序型 有内在排序，可以比较，但是不适合加减运算。如年级编号。
* continuous 连续型  数值

**dataType**：
字段值的类型（这个更类似于Java 或 C中的数据类型），比如类别型，就是所有的string
可选值如下：
* string
* integer
* float
* double
* boolean
* date
* time
* dateTime
* dateDaysSince[0]
* dateDaysSince[1960]
* dateDaysSince[1970]
* dateDaysSince[1980]
* timeSeconds
* dateTimeSecondsSince[0]
* dateTimeSecondsSince[1960]
* dateTimeSecondsSince[1970]
* dateTimeSecondsSince[1980]

类型的定义，详见[XML Schema Part 2: Datatypes](https://www.w3.org/TR/xmlschema-2/)
其中有三个类型是PMML 相较于 XML Schema 新增的，分别是timeSeconds,dateDaysSince[a Year],dateTimesSecondsSince[a Year],其中a Year 可以是0、1960、1970、1980中的一个值。注意到，a Year不是一个任意值，必须在允许的值中进行选取。如果可变的a Year 是必须的，那就需要考虑使用[內建函数](http://dmg.org/pmml/v4-3/BuiltinFunctions.html#datedayssinceyear)  
PMML支持这些额外的类型，原因在于在数据挖掘中，经常需要将时间转化为可以比较和计算的数值。例如 2003-04-01 就可以通过 dateDaysSince[1960] 转为15796。


**taxonomy**：
可选属性 **taxonomy** 关联了一个[taxonomy](https://en.wikipedia.org/wiki/Taxonomy) 值。用来描述一个分层的值的情况。  
*Tips*：
这个值体现了一个继承性和分层性。

**isCyclic**:
表示定序型的值是否可循环，如周日至周六

# 有效性
DataFiled 定义了一组规则来验证值的有效性。
数据挖掘模型区分了如下三种值的情况：  
* Missing Data： 缺失值。  
* Invalid Value： 无效值，即输入不为空，但是取值范围超出了给该值的取值区间。
* Valid Value： 有效值，即不为以上两种情况的值。

# 值和区间
Value 和 Interval 元素用来描述数据字典中值的情况和取值区间。

## Value Schema
```
<xs:element name="Value">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="value" type="xs:string" use="required"/>
    <xs:attribute name="displayValue" type="xs:string"/>
    <xs:attribute name="property" default="valid">
      <xs:simpleType>
        <xs:restriction base="xs:string">
          <xs:enumeration value="valid"/>
          <xs:enumeration value="invalid"/>
          <xs:enumeration value="missing"/>
        </xs:restriction>
      </xs:simpleType>
    </xs:attribute>
  </xs:complexType>
</xs:element>
```



## Interval Schema
```
<xs:element name="Interval">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="closure" use="required">
      <xs:simpleType>
        <xs:restriction base="xs:string">
          <xs:enumeration value="openClosed"/>
          <xs:enumeration value="openOpen"/>
          <xs:enumeration value="closedOpen"/>
          <xs:enumeration value="closedClosed"/>
        </xs:restriction>
      </xs:simpleType>
    </xs:attribute>
    <xs:attribute name="leftMargin" type="NUMBER"/>
    <xs:attribute name="rightMargin" type="NUMBER"/>
  </xs:complexType>
</xs:element>

```

## 连续型demo
```
<DataField name="SomeVariable" dataType="double" optype="continuous">
  <Interval closure="closedClosed" leftMargin="0" rightMargin="100"/>
  <Value property="missing" value="-999"/>
</DataField>
```
表示取值区间[0, 100]，若为空，则取-999
Interval 只支持连续型（continuous)

## 类别型demo
```
<DataField name="_target" optype="categorical" dataType="string">
	<Value value="0"/>
	<Value value="1"/>
</DataField>
```
不存在二分类的数据类型，所以二分类的定义如上。

## 定序型demo
```
<DataField name="Volume" optype="ordinal" dataType="string">
  <Value value="loud"/>
  <Value value="louder"/>
  <Value value="insane"/>
</DataField>
```
如上，定序为出现顺序的逆序，及loud < louder < insane
加上 cyclic 参数，即可形成循环，如周日至周六




# REF
[PMML 4.3 - Data Dictionary](http://dmg.org/pmml/v4-3/DataDictionary.html)