# 说明
* 基于PMML Version4.3
* 基本可以视为官方文档的一个翻译
* 如有疏漏，请联系yao544303963@gmail.com

# 转换字典以及衍生字段
大多数情况，数据挖掘模型使用了一些简单的函数，将输入数据转换为模型能够简单使用的值。例如，神经网络内部大多使用[0,1]范围内的实数进行计算。数值型输入通常转换到[0,1]区间内，类别型输入，通过one-hot编码，哑变量等方式，处理为0/1串。

PMML定义了如下的转换操作:
* 标准化：也有叫归一化的，将值银蛇为数字。
* 离散化：将连续值离散化。
* 值映射：将一组离散值映射为一组离散值。
* 文本编索引：为现有项编号。
* 自定义函数
* 聚合：求和或计算均值
* Lag：加后缀

这一类操作的XML元素，都由DerivedField 标记围绕，DerivedField 为这些映射提供了一个通用的元素。同一个模型可以定义多个这种转换。转换之后的字段有一个名字，模型可以通过这个名字找到。

PMML中支持的转换操作并不能覆盖所有数据挖掘中有可能使用到的转换操作。相对的，这些转化操作大多由某些数据挖掘系统自动产生，例如在在神经网络中，对输入标准化。又如，为了计算分位范围。

# Sechema
```
<xs:group name="EXPRESSION">
  <xs:choice>
    <xs:element ref="Constant"/>
    <xs:element ref="FieldRef"/>
    <xs:element ref="NormContinuous"/>
    <xs:element ref="NormDiscrete"/>
    <xs:element ref="Discretize"/>
    <xs:element ref="MapValues"/>
    <xs:element ref="TextIndex"/>        
    <xs:element ref="Apply"/>
    <xs:element ref="Aggregate"/>
    <xs:element ref="Lag"/>
  </xs:choice>
</xs:group>

<xs:element name="TransformationDictionary">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element ref="DefineFunction" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element ref="DerivedField" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>

<xs:element name="LocalTransformations">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element ref="DerivedField" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>

<xs:element name="DerivedField">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:group ref="EXPRESSION"/>
      <xs:element ref="Value" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="name" type="FIELD-NAME"/>
    <xs:attribute name="displayName" type="xs:string"/>
    <xs:attribute name="optype" type="OPTYPE" use="required"/>
    <xs:attribute name="dataType" type="DATATYPE" use="required"/>
  </xs:complexType>
</xs:element>
```

# 说明
如果一个 DerivedField 是 TransformationDictionary 或者 LocalTransformations的子元素，那么name属性是必要的。
如果DerivedField 是内联的，（即不是 TransformationDictionary 或者 LocalTransformations的子元素），那么name属性是可选的。
TransformationDictionary 元素，允许这些转换操作定义一次而被该PMML模型的任何元素使用。相关操作见[Scope of Fileds](http://dmg.org/pmml/v4-3/FieldScope.html)
DerivedField中的EXPRESSION定义了转换操作如何进行。
因为结果类型是未知的，因而optype属性是必须的，例如一个如下映射
"cat" -> "0.1"  
"dog" -> "0.2"  
"elephat" -> "0.3"
对于0.1 不注明的话，无法确定是一个string还是一个number，因而需要optype进行注明。

A DerivedField may have a list of Value elements that define the ordering of the values for an ordinal field. The attribute property must not be used for Value elements within a DerivedField. That is, the list cannot specify values that are interpreted as missing or invalid.


## Constant
```
<xs:element name="Constant">
  <xs:complexType>
    <xs:simpleContent>
      <xs:extension base="xs:string">
        <xs:attribute name="dataType" type="DATATYPE"/>
      </xs:extension>
    </xs:simpleContent>
  </xs:complexType>
</xs:element>
```
定义一个常量，Constant可以用在有多个入参的表达式中。常量的实际值，由元素内容获得，如 <Constant>1.05</Constant> 代表了数值 1.05。常量的dataType 属性是可选的。如果ParameterField 定义中包括dataType，那么Constant中的dataType会继承parameterField中的定义。如果没有定义，则会根据内容进行推断，如整数会推断为integer，小数会推断为float，非数值会推断为string，复杂类型的dataType会在[Function](http://dmg.org/pmml/v4-3/Functions.html)中指定。

## FieldRef
```
<xs:element name="FieldRef">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="field" type="FIELD-NAME" use="required"/>
    <xs:attribute name="mapMissingTo" type="xs:string"/>
  </xs:complexType>
</xs:element>
```
用来描述那些在转换操作中例外的字段，如10个字段中9个字段需要进行标准化，1个不需要。
当输入项是缺失时，默认的输出也是缺失。定义了mapMissingTo，就为缺失值指定一个默认值。

## Normalization
这个元素提供了一个基础的框架，来将输入值映射到一个指定的区间。通常为[0, 1]。

### NormContinuous
```
<xs:element name="NormContinuous">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element ref="LinearNorm" minOccurs="2" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="mapMissingTo" type="NUMBER"/>
    <xs:attribute name="field" type="FIELD-NAME" use="required"/>
    <xs:attribute name="outliers" type="OUTLIER-TREATMENT-METHOD" default="asIs"/>
  </xs:complexType>
</xs:element>

<xs:element name="LinearNorm">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="orig" type="NUMBER" use="required"/>
    <xs:attribute name="norm" type="NUMBER" use="required"/>
  </xs:complexType>
</xs:element>
```
通过一个分段线性函数，将输入进行映射操作。
LinerNorm 元素定义了用于转化的线性函数的序列。函数序列至少包含两个元素（minOccurs="2"），并且所有函数的排序必须和属性orig值的升序保持一致。示例如下图：
![](http://dmg.org/pmml/v4-3/Transformations_InterpolationA_4x3.gif)
给出两个分段点(a1,b1),(a2,b2)。并且不存在一个新的分段点(a3,b3)，使得a1<\a3<\a2。（图中，a1应该是个分段点，但是和小于a1的斜率区分不明显）
这样就有
b1+ ( x-a1)/(a2-a1)*(b2-b1) for a1 ≤ x ≤ a2。

缺失的输入在没有定义的情况下，会产生一个缺失的输出。
如果输入超出了范围[a1 .. an]，则做为异常值处理。类似缺失值，异常值有如下三种处理方式：

* asIs：通过最相近的区间来计算
* asMissingValues：当成缺失值处理
* asExtremeValues：当成区间极值处理。情况如下图，当值小于a1 或者大于 a3时，映射值和b1、b3一样。
![](http://dmg.org/pmml/v4-3/Transformations_InterpolationE_4x3.gif)

NormContinuous 也能实现[z-score](https://baike.baidu.com/item/Z%E5%88%86%E6%95%B0/8268473?fr=aladdin)标准化
样例如下：
```
<NormContinuous field="X">
  <LinearNorm orig="0" norm="-m/s"/>
  <LinearNorm orig="m" norm="0"/>
</NormContinuous>
```

### Normalize discrete values
```
<xs:element name="NormDiscrete">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="field" type="FIELD-NAME" use="required"/>    
    <xs:attribute name="value" type="xs:string" use="required"/>
    <xs:attribute name="mapMissingTo" type="NUMBER"/>
  </xs:complexType>
</xs:element>
```
许多数据挖掘模型为了对字符串进行数学计算，会将字符串值编码为数字值。比如[哑变量](https://baike.baidu.com/item/%E8%99%9A%E6%8B%9F%E5%8F%98%E9%87%8F?fromtitle=%E5%93%91%E5%8F%98%E9%87%8F&fromid=9621990)处理。PMML中使用NormDiscrete来处理这一类型的操作。

定义了一个元素(f,v),如果f的值为v，则返回1否则为0。


## Discretization
```
<xs:element name="Discretize">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element ref="DiscretizeBin" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="field" type="FIELD-NAME" use="required"/>
    <xs:attribute name="mapMissingTo" type="xs:string"/>
    <xs:attribute name="defaultValue" type="xs:string"/>
    <xs:attribute name="dataType" type="DATATYPE"/>
  </xs:complexType>
</xs:element>

<xs:element name="DiscretizeBin">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element ref="Interval"/>
    </xs:sequence>
    <xs:attribute name="binValue" type="xs:string" use="required"/>
  </xs:complexType>
</xs:element>
```
离散化操作是根据给出的区间，将输入的连续值划分到不同的区间中。
**field**：  
该属性定义了输入字段名。
**DiscretizeBin**：  
该元素定义了将属于区间i的值映射为分箱i。
离散化操作也制造一个缺失值的输出，这种情况下，DiscretizeBin是无效的，可以不用定义。

### 操作表
![image](https://upload-images.jianshu.io/upload_images/3698622-bb72a7ad5bbf6ca4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
情况如上图所示，以第五行为例，如输入缺失，有没有指定ampMissingTo，则输出也为缺失。

### 样例
```
<Discretize field="Profit">
  <DiscretizeBin binValue="negative">
    <Interval closure="openOpen" rightMargin="0"/>
    <!-- left margin is -infinity by default -->
  </DiscretizeBin>
  <DiscretizeBin binValue="positive">
    <Interval closure="closedOpen" leftMargin="0"/>
    <!-- right margin is +infinity by default -->
  </DiscretizeBin>
</Discretize>
```
输入是Profit字段，大于0为positive，小于0为negative

## Map Values
```
<xs:element name="MapValues">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element minOccurs="0" maxOccurs="unbounded" ref="FieldColumnPair"/>
      <xs:choice minOccurs="0">
        <xs:element ref="TableLocator"/>
        <xs:element ref="InlineTable"/>
      </xs:choice>
    </xs:sequence>
    <xs:attribute name="mapMissingTo" type="xs:string"/>
    <xs:attribute name="defaultValue" type="xs:string"/>
    <xs:attribute name="outputColumn" type="xs:string" use="required"/>
    <xs:attribute name="dataType" type="DATATYPE"/>
  </xs:complexType>
</xs:element>

<xs:element name="FieldColumnPair">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="field" type="FIELD-NAME" use="required"/>
    <xs:attribute name="column" type="xs:string" use="required"/>
  </xs:complexType>
</xs:element>
```
只要列出映射关系对，任意离散值都可以被映射为其他的离散值。这个映射关系对列表通过表实现，这个表可以在PMML内用XML定义，也可以是一个外部表（类似Taxonomy的实现，因为表实在太大了）。

其中 **InlineTable** 和 **TableLocator**元素在[Taxonomy schema.](http://dmg.org/pmml/v4-3/Taxonomy.html)中定义。
mapMissingTo 属性定义和前文相同


### 操作表
![](https://upload-images.jianshu.io/upload_images/3698622-29da28d5f4359798.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 不同的值可以映射为同一个值（即Map中，不同的Key，Value可以相同）
* 映射关系中的key存在重复会报错（如 1->2, 1->3 这种情况）
* 如果key不在map中，结果为缺失。

### 样例1
```
<MapValues outputColumn="longForm">
  <FieldColumnPair field="gender" column="shortForm"/>
  <InlineTable>
    <row><shortForm>m</shortForm><longForm>male</longForm>
    </row>
    <row><shortForm>f</shortForm><longForm>female</longForm>
    </row>
  </InlineTable>
</MapValues>
```
将m映射为male，将f映射为female。类似如下SQL
```
CASE "gender" When 'm' Then 'male' When 'f' Then 'female' End
```

### 样例2
```
<DerivedField dataType="double" optype="continuous">
  <MapValues outputColumn="out" dataType="integer">
    <FieldColumnPair field="BAND" column="band"/> 
    <FieldColumnPair field="STATE" column="state"/> 
    <InlineTable>
      <row>
        <band>1</band> 
        <state>MN</state> 
        <out>10000</out> 
      </row>
      <row>
        <band>1</band> 
        <state>IL</state> 
        <out>12000</out> 
      </row>
      <row>
        <band>1</band> 
        <state>NY</state> 
        <out>20000</out> 
      </row>
      <row>
        <band>2</band> 
        <state>MN</state> 
        <out>20000</out> 
      </row>
      <row>
        <band>2</band> 
        <state>IL</state> 
        <out>23000</out> 
      </row>
      <row>
        <band>2</band> 
        <state>NY</state> 
        <out>30000</out> 
      </row>
    </InlineTable>
  </MapValues>
</DerivedField>
```

多维映射的情况，如在NY、band1的薪水为20,000

state	band 1	band 2
MN	   10,000	20,000
IL	   12,000	23,000
NY	   20,000	30,000

### 样例3
```
<DerivedField name="LSTROPEN_MIS" optype="categorical" dataType="string">
  <MapValues mapMissingTo="Missing" defaultValue="Not missing" outputColumn="none">
    <FieldColumnPair field="LSTROPEN" column="none"/>
  </MapValues>
</DerivedField>
```
可以使用映射来解决值缺失问题。

## Extracting term frequencies from text
```
<xs:element name="TextIndex">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element ref="TextIndexNormalization" minOccurs="0" maxOccurs="unbounded"/>
      <xs:group ref="EXPRESSION"/>
    </xs:sequence>
    <xs:attribute name="textField" type="FIELD-NAME" use="required"/>
    <xs:attribute name="localTermWeights" default="termFrequency">
      <xs:simpleType>
        <xs:restriction base="xs:string">
          <xs:enumeration value="termFrequency"/>
          <xs:enumeration value="binary"/>
          <xs:enumeration value="logarithmic"/>
          <xs:enumeration value="augmentedNormalizedTermFrequency"/>
        </xs:restriction>
      </xs:simpleType>
    </xs:attribute>
    <xs:attribute name="isCaseSensitive" type="xs:boolean" default="false"/>
    <xs:attribute name="maxLevenshteinDistance" type="xs:integer" default="0"/>
    <xs:attribute name="countHits" default="allHits">
      <xs:simpleType>
        <xs:restriction base="xs:string">
          <xs:enumeration value="allHits"/>
          <xs:enumeration value="bestHits"/>
        </xs:restriction>
      </xs:simpleType>
    </xs:attribute>
    <xs:attribute name="wordSeparatorCharacterRE" type="xs:string" default="\s"/>
    <xs:attribute name="tokenize" type="xs:boolean" default="true"/>    
  </xs:complexType>
</xs:element>
```
为了能够在一个PMML模型中量化文本输入，可以使用TextIndex操作。TextIndex 元素中定义了怎样给输入文本项编写索引，包括案例驱动，标准化以及其他设定。这是一个单独的表达元素，值通常用一个常量表示。

TextIndex 的转化，主要根据词频来计算，计算规则由**localTermWrights** 属性设定。计算规则[详细描述见此](http://www.ca.sandia.gov/~tgkolda/pubs/bibtgkfiles/umcp-cs-tr-3724.pdf)，简述如下：
* termFrequency： 使用出现词频，即x = freq
* binary： 出现则为1，没出现则为0，即 x = χ(freq)
* logarithmic：计算词频的log（10为底）值，即 x=log(1 + freq)
* augmentedNormalizedTermFrequency：计算一个加权值，公式如 x = 0.5 * (χ(freqi) + (freq / maxk(freqk)))

**isCaseSensitive** 属性定义了文本输入需要明确匹配要求时，才会返回频次。
**maxLevenshteinDistance** 

TODO



## Aggregations
```
<xs:element name="Aggregate">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="field" type="FIELD-NAME" use="required"/>
    <xs:attribute name="function" use="required">
      <xs:simpleType>
        <xs:restriction base="xs:string">
          <xs:enumeration value="count"/>
          <xs:enumeration value="sum"/>
          <xs:enumeration value="average"/>
          <xs:enumeration value="min"/>
          <xs:enumeration value="max"/>
          <xs:enumeration value="multiset"/>
        </xs:restriction>
      </xs:simpleType>
    </xs:attribute>
    <xs:attribute name="groupField" type="FIELD-NAME"/>
    <xs:attribute name="sqlWhere" type="xs:string"/>
  </xs:complexType>
</xs:element>
```
类似于SQL中的GOURP BY操作，对字段根据值来进行操作。
一个例子如下：
```
<Aggregate field="item" function="multiset" groupField="transaction"/>
```
对transaction进行group by，对每个transaction生成一个item 的list。

## Lag
```
<xs:element name="Lag">
    <xs:complexType>
      <xs:sequence>
	<xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
	<xs:element ref="BlockIndicator" minOccurs="0" maxOccurs="unbounded"/>
      </xs:sequence>
      <xs:attribute name="field" type="FIELD-NAME" use="required"/>
      <xs:attribute name="n" type="xs:positiveInteger" default="1"/>
  </xs:complexType>
  </xs:element>

  
<xs:element name="BlockIndicator">
    <xs:complexType>
      <xs:attribute name="field" type="FIELD-NAME" use="required"/>
    </xs:complexType>
  </xs:element>
```
定义了


# REF
[PMML 4.3 - Transformation Dictionary and Derived Fields](http://dmg.org/pmml/v4-3/Transformations.html)