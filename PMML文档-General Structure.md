# 说明
* 基于PMML Version4.3
* 基本可以视为官方文档的一个翻译
* 如有疏漏，请联系yao544303963@gmail.com

# 基本结构
PMML使用XML来表示数据挖掘模型。模型的结构用一个XML Schema来描述。一个PMML文档中可以包含多个数据挖掘模型。一个PMML文档本质就是一个XML文档，其根元素是一个PMML类型的元素。其基本结构如下：
```
<?xml version="1.0"?>
<PMML version="4.3"
  xmlns="http://www.dmg.org/PMML-4_3" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <Header copyright="Example.com"/>
  <DataDictionary> ... </DataDictionary>

  ... a model ...

</PMML>
```

命名空间定义如下:
```
  <xs:schema
    xmlns:xs="http://www.w3.org/2001/XMLSchema"
    targetNamespace="http://www.dmg.org/PMML-4_3"
    xmlns="http://www.dmg.org/PMML-4_3"
    elementFormDefault="unqualified">
```
这里需要注意，因为命名空间在当前形式中声明，所以PMML中不能使用不同的命名空间。
关于命名空间，详见[XML 命名空间](http://www.w3school.com.cn/xml/xml_namespaces.asp)

尽管一个PMML文件必须严格符合PMML XSD的定义，然而并不需要额外的解析器。为了确保是一个有效的XML文档，PMML 需要遵循一些额外的规则。更多的内容参见[conformance rules ](http://dmg.org/pmml/v4-3/Conformance.html)

PMML文档的根元素，必须是PMML 类型的。如下：
```
<xs:element name="PMML">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Header"/>
      <xs:element ref="MiningBuildTask" minOccurs="0"/>
      <xs:element ref="DataDictionary"/>
      <xs:element ref="TransformationDictionary" minOccurs="0"/>
      <xs:sequence minOccurs="0" maxOccurs="unbounded">
        <xs:group ref="MODEL-ELEMENT"/>
      </xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="version" type="xs:string" use="required"/>
  </xs:complexType>
</xs:element>

<xs:group name="MODEL-ELEMENT">
  <xs:choice>
    <xs:element ref="AssociationModel"/>
    <xs:element ref="BayesianNetworkModel"/>
    <xs:element ref="BaselineModel"/>
    <xs:element ref="ClusteringModel"/>
    <xs:element ref="GaussianProcessModel"/>
    <xs:element ref="GeneralRegressionModel"/>
    <xs:element ref="MiningModel"/>
    <xs:element ref="NaiveBayesModel"/>
    <xs:element ref="NearestNeighborModel"/>
    <xs:element ref="NeuralNetwork"/>
    <xs:element ref="RegressionModel"/>
    <xs:element ref="RuleSetModel"/>
    <xs:element ref="SequenceModel"/>
    <xs:element ref="Scorecard"/>
    <xs:element ref="SupportVectorMachineModel"/>
    <xs:element ref="TextModel"/>
    <xs:element ref="TimeSeriesModel"/>
    <xs:element ref="TreeModel"/>
  </xs:choice>
</xs:group>
```

其中，模型部分由如下定义
```
<xs:sequence minOccurs="0" maxOccurs="unbounded">
    <xs:group ref="MODEL-ELEMENT"/>
</xs:sequence>
```
可以看出，一个PMML文档可以包含不止一个模型。如果应用系统提供了模型的命名，并且PMML的消费者指定了该模型的命名。否则会使用第一个模型。
minOccurs="0",这个参数表示，模型列表可以为空。模型为空时，表示该PMML用来传输原始数据。不包含模型的PMML对于PMML消费者而言，意义不大。  

对于PMML4.3，属性**version**的值，必须为4.3。

元素MiningBuildTask 包含生成该模型时及训练阶段的配置参数的描述。该信息不是必要的，但许多情况下，对于模型的维护和理解很有帮助。MiningBuildTask中特有的结构，不是由PMML定义的。示例如下：
```
<xs:element name="MiningBuildTask">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>
```


通常来讲，PMML中的字段名是独一无二的。避免字段名重复是一个好习惯，可以使消费者使用起来更为简便，减少异常的发生。

某一类型的PMML模型都有其特定的用途，例如神经网络或者逻辑回归。有的用于回归，有的用于分类。因此PMML定义了5种不同类型的MINING-FUNCTION，如下：
```
<xs:simpleType name="MINING-FUNCTION">
  <xs:restriction base="xs:string">
    <xs:enumeration value="associationRules"/>
    <xs:enumeration value="sequences"/>
    <xs:enumeration value="classification"/>
    <xs:enumeration value="regression"/>
    <xs:enumeration value="clustering"/>
    <xs:enumeration value="timeSeries"/>
    <xs:enumeration value="mixed"/>
  </xs:restriction>
</xs:simpleType>
```
分别为关联规则，序列，分类，聚类，回归。时间序列和混合，是上述5种的组合。

所有的PMML模型的顶层元素，都类似如下结构
```
 <xs:element name="ExampleModel">
    <xs:complexType>
      <xs:sequence>
        <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
        <xs:element ref="MiningSchema"/>
        <xs:element ref="Output" minOccurs="0"/>
        <xs:element ref="ModelStats" minOccurs="0"/>
        <xs:element ref="Targets" minOccurs="0"/>
        <xs:element ref="LocalTransformations" minOccurs="0" />
        ...
        <xs:element ref="ModelVerification" minOccurs="0"/>
        <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      </xs:sequence>
      <xs:attribute name="modelName" type="xs:string" use="optional"/>
      <xs:attribute name="functionName" type="MINING-FUNCTION" use="required"/>
      <xs:attribute name="algorithmName" type="xs:string" use="optional"/>
    </xs:complexType>
  </xs:element>
```
MiningSchema 是一个由模型使用到的字段组成的非空列表。
Output 模型的计算结果，比如预测概率或置信度等。
ModelStats 是字段的一个统计属性。
Targets 包含了目标值，以及相关的信息，如 0 1分类中，属于各个分类的概率。
LocalTransformations 包含了在本地转换操作用到的追加字段。
...(其他一些元素的定义)
ModelVerification 给出了样例数据以及模型结果样例，便于消费者验证有效性。

**属性**  
**modelName** 属性代表模型名字，用来确保该模型在PMML文件中的唯一性。该字段不是必要的。消费者可以根据需要，自由的管理模型名

**functionName** 和 **algorithmName** 描述了数据挖掘模型的类型，例如是聚类模型还是分类模型。**algorithmName** 是自由类型的且只用于说明，可以包含任何用于描述该该算法和模型的内容。

# 约束
尽管非常罕见，分类模型可能出现多个解的情况。在这些情况下，PMML没有定义特殊的处理流程，只是按照DataFiled 中对类别的排序，推荐先出现的类别。

# 命名规范
PMML的命名规范如下：
* 元素名，首字母大写
* 属性名，首字母小写
* 常量，首字母小写
* 简单类型，全部大写

为避免和减号产生歧义，'-'不建议使用。

# 扩展机制
在PMML Schema中可以通过Extension元素扩展模型内容。Extension元素必须出现在第一个子元素，或者组中第一的位置。这为了保证该元素中接下来的内容都受Extension元素的影响。每一个主要的元素都必须包含Extension元素作为第一个和最后一个子元素，来确保其最大的扩展性。

Extension Schema如下：
```
<xs:element name="Extension">
  <xs:complexType>
    <xs:complexContent mixed="true">
      <xs:restriction base="xs:anyType">
        <xs:sequence>
          <xs:any processContents="skip" minOccurs="0" maxOccurs="unbounded"/>
        </xs:sequence> 
        <xs:attribute name="extender" type="xs:string" use="optional"/>
        <xs:attribute name="name" type="xs:string" use="optional"/>
        <xs:attribute name="value" type="xs:string" use="optional"/>
      </xs:restriction>
    </xs:complexContent>
  </xs:complexType>
</xs:element>
```

这些扩展的元素都含有一个ANY类型的模型来指向扩展的模型。不过这些元素的类型名必须以X开头，来避免和PMML后续增加的内容产生冲突。
相应的，Extension元素也有**name** 和 **value** 属性来指定字段名和值。

## 样例
扩展属性format可以通过如下方式在DataField内追加：
```
<DataField name="foo" dataType="double" optype="continuous">
  <Extension name="format" value="%9.2f"/>
</DataField>
```

扩展元素 DataFieldSource 可以通过如下方式追加到DataField中
```
<DataField name="foo" dataType="double" optype="continuous">
  <Extension>
    <DataFieldSource sourceKnown="yes">
      <Source>derivedFromInput</Source>
    </DataFieldSource>
  </Extension>
</DataField>
```

# 基础数据类型和对象
```
<xs:simpleType name="NUMBER">
  <xs:restriction base="xs:double">
  </xs:restriction>
</xs:simpleType>
```
该定义常用来区别数值和其他类型的值。数值通常会包含符号，小数点，指数。XML Schema中float类型支持INF，-INF 和 NaN的表示。这些在NUMBER中是不支持的。除了NUMBER还有一些定义更严格的类型，类似于NUMBER的子类型。

**INT-NUMBER** 
```
<xs:simpleType name="INT-NUMBER">
  <xs:restriction base="xs:integer">
  </xs:restriction>
</xs:simpleType>
```
必须是整数，不能是小数或指数。

**REAL-NUMBER**  
```
<xs:simpleType name="REAL-NUMBER">
  <xs:restriction base="xs:double">
  </xs:restriction>
</xs:simpleType>
```
实数型，可以覆盖C/C++中 float、long或者double类型。科学计数法，如 1，23e4 也是支持的。INF,-INF，NaN 是不支持的。

**PROB-NUMBER**  
```
<xs:simpleType name="PROB-NUMBER">
  <xs:restriction base="xs:double">
  </xs:restriction>
</xs:simpleType>
```
概率型，值在[0.0,1.0]之间，通常用来表示预测概率。

**PERCENTAGE-NUMBER**  
```
<xs:simpleType name="PERCENTAGE-NUMBER">
  <xs:restriction base="xs:double">
  </xs:restriction>
</xs:simpleType>
```
百分比型，[0.0,100.0]之间的一个实数。


注意到这些对象并不强制XML解析器来检查他们的数据类型。但在PMML文档中定义了其有效性校验的相关操作。
许多元素都有输入字段的引用。PMML不使用[IDERF](https://blog.csdn.net/sweatlove/article/details/1681648)来表示字段名，就是因为XML 标识的有效性校验不是必须的。而通过如下定义:
```
<xs:simpleType name="FIELD-NAME">
  <xs:restriction base="xs:string">
  </xs:restriction>
</xs:simpleType>
```
则可以进行相关引用的校验。

# 简单数组
模型中通常包含一大堆数值组成的集合。Array 的定义和C or Java中的数组类似，Schema如下：
```
<xs:complexType name="ArrayType" mixed="true">
  <xs:attribute name="n" type="INT-NUMBER" use="optional"/>
  <xs:attribute name="type" use="required">
    <xs:simpleType>
      <xs:restriction base="xs:string">
        <xs:enumeration value="int"/>
        <xs:enumeration value="real"/>
        <xs:enumeration value="string"/>
      </xs:restriction>
    </xs:simpleType>
  </xs:attribute>
</xs:complexType>

<xs:element name="Array" type="ArrayType"/>
```

Array 的内容是一组以空格分割的值，多个空格和单个空格效果一致。  
**n**：该属性定义了该序列的值数量。如果**n**的值给出了，那和值的数量必须保持一致，否则该pmml文档视为无效。
**type**：该属性是必须的，指定类型便于后续解析。
特殊的，在String中存在空格 和 " ,需要使用转义符，例子如下：
**例子**
```
<Array n="3" type="int">1 22 3</Array>
<Array n="3" type="string">ab  "a b"   "with \"quotes\" "</Array>
```

类似上面NUMBER有很多子类型，Array 也有，如下：
```
<xs:group name="NUM-ARRAY">
  <xs:choice>
    <xs:element ref="Array"/>
  </xs:choice>
</xs:group>

<xs:group name="INT-ARRAY">
  <xs:choice>
    <xs:element ref="Array"/>
  </xs:choice>
</xs:group>

<xs:group name="REAL-ARRAY">
  <xs:choice>
    <xs:element ref="Array"/>
  </xs:choice>
</xs:group>

<xs:group name="STRING-ARRAY">
  <xs:choice>
    <xs:element ref="Array"/>
  </xs:choice>
</xs:group>
```

# 稀疏数组
只记录非0值的数组，类似系数矩阵
```
<xs:element name="INT-SparseArray">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Indices" minOccurs="0"/>
      <xs:element ref="INT-Entries" minOccurs="0"/>
    </xs:sequence>
    <xs:attribute name="n" type="INT-NUMBER" use="optional"/>
    <xs:attribute name="defaultValue" type="INT-NUMBER" use="optional" default="0"/>
  </xs:complexType>
</xs:element>

<xs:element name="REAL-SparseArray">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Indices" minOccurs="0"/>
      <xs:element ref="REAL-Entries" minOccurs="0"/>
    </xs:sequence>
    <xs:attribute name="n" type="INT-NUMBER" use="optional"/>
    <xs:attribute name="defaultValue" type="REAL-NUMBER" use="optional" default="0"/>
  </xs:complexType>
</xs:element>

<xs:element name="Indices">
  <xs:simpleType>
    <xs:list itemType="xs:int"/>
  </xs:simpleType>
</xs:element>

<xs:element name="INT-Entries">
  <xs:simpleType>
    <xs:list itemType="xs:int"/>
  </xs:simpleType>
</xs:element>

<xs:element name="REAL-Entries">
  <xs:simpleType>
    <xs:list itemType="xs:double"/>
  </xs:simpleType>
</xs:element>
```

**n** 该属性指定了稀疏数组的长度，这在没有指定数组最后一个对象的情况下，会非常有用。
**defaultValue** 该属性指定了整个数组中，没有指定值情况下的默认值。
稀疏数组其实是由两个数组组成，索引数组Indices和值数组INT-Entries 或者 REAL-Entries。
索引数组从1开始，包含了值不为默认值的索引。
值数组是索引数组对应编号位的值。因此索引数组和值数组的长度必须保持一致。
形成稀疏数组的结构  如 1:3 2:6 8:9的索引数组为[1,2,8] 值数组为[3,6,9]。
综上，索引数组和值数组是成对出现的，否则PMML会视为无效。

## 样例
数组 0 3 0 0 42 0 0 可以用如下形式表示
```
<INT-SparseArray n="7">
  <Indices>2 5</Indices>
  <INT-Entries>3 42</INT-Entries>
</INT-SparseArray>
```

数组 0 0 0 0 0 0 0 可以用如下形式表示
```
<INT-SparseArray n="7"/>
```

# 矩阵
为了节省空间，一个矩阵会以对角矩阵或者稀疏矩阵的形式存储。
```
<xs:element name="Matrix">
  <xs:complexType>
    <xs:choice minOccurs="0">
      <xs:group ref="NUM-ARRAY" maxOccurs="unbounded"/>
      <xs:element ref="MatCell" maxOccurs="unbounded"/>
    </xs:choice>
    <xs:attribute name="kind" use="optional" default="any">
      <xs:simpleType>
        <xs:restriction base="xs:string">
          <xs:enumeration value="diagonal"/>
          <xs:enumeration value="symmetric"/>
          <xs:enumeration value="any"/>
        </xs:restriction>
      </xs:simpleType>
    </xs:attribute>
    <xs:attribute name="nbRows" type="INT-NUMBER" use="optional"/>
    <xs:attribute name="nbCols" type="INT-NUMBER" use="optional"/>
    <xs:attribute name="diagDefault" type="REAL-NUMBER" use="optional"/>
    <xs:attribute name="offDiagDefault" type="REAL-NUMBER" use="optional"/>
  </xs:complexType>
</xs:element>

<xs:element name="MatCell">
  <xs:complexType>
    <xs:simpleContent>
      <xs:extension base="xs:string">
        <xs:attribute name="row" type="INT-NUMBER" use="required"/>
        <xs:attribute name="col" type="INT-NUMBER" use="required"/>
      </xs:extension>
    </xs:simpleContent>
  </xs:complexType>
</xs:element>
```

矩阵通常用一个数组的序列（二维数组）或者矩阵元素的序列来表示。如果用数组表示，每个数组表示矩阵的一行。
**MatCells** 矩阵元素通过行、列索引来指定对应位置的值。行、列索引从1开始。
**diagDefault** 和 **offDiagDefault** 属性在使用稀疏矩阵时，必须设定。
**nbRows** 和 **nbCols** 属性指定了矩阵的维度。如果其中一个属性没有指定，那便有矩阵表示（数组或者矩阵元素）的定义隐式定义。如使用了矩阵元素，那么矩阵的维度就是能容纳该元素得的维度值。

**kind** 属性触发了矩阵的实际表示：
* diagonal: 只有一个数组，用来表示矩阵对角线元素
* symmetric: 对称矩阵，用二维数组表示，第一个数组表示坐标(0,0)处元素, 第二个数组表示坐标(1，0)和(1，1)处元素，以此类推。描述整个左下半三角区。右上半三角区由对称求得。
* any: 通过二维数组或者矩阵元素指定矩阵。

## 样例
矩阵
```
0  0  0 42  0
0  1  0  0  0
5  0  0  0  0
0  0  0  0  7
0  0  9  0  0
```
可以用如下定义来表述：
```
<Matrix nbRows="5" nbCols="5">
  <Array type="real">0 0 0 42 0</Array>
  <Array type="real">0 1 0 0 0</Array>
  <Array type="real">5 0 0 0 0</Array>
  <Array type="real">0 0 0 0 7</Array>
  <Array type="real">0 0 9 0 0</Array>
</Matrix>
<Matrix diagDefault="0" offDiagDefault="0">
  <MatCell row="1" col="4">42</MatCell>
  <MatCell row="2" col="2">1</MatCell>
  <MatCell row="3" col="1">5</MatCell>
  <MatCell row="4" col="5">7</MatCell>
  <MatCell row="5" col="3">9</MatCell>
</Matrix>
```

# 无评分模型
找寻一个有效模型的道路通常是曲折和充满荆棘的。生成可部署的模型的尝试中出现失败是很正常的。在早期的试验阶段，会进行非常多的测试来寻找有用的特征。或者，从基础的角度说，训练模型使用的数据有问题的话，最后也无法生成正确的模型。相对，许多数据挖掘工具都会自动排除那些不符合要求的特征，或者制定一个最低限度的标准来确保模型可以被部署。

PMML包含许多帮助使用者了解模型质量的特征性，比如Statics 和 Model Explanation。这些描述性的元素对于验证模型的有效性，以及理解模型运行失败的原因很有包住。

反过来看，PMML既能产生好的结果，也能产生坏的结果，尤其是在整个系统中，PMML只是承担了接口人的角色，联接了模型生成者和模型消费者。这也就要求模型的消费者能够辨别PMML中是否包含一个有效的模型，或者PMML不能用于判断得分。
举个例子，如回归模型中，所有的独立变量都不能满足最小的重要性验证。在追加的描述信息中，应该有所体现，并且说明为何这些变量会被排除。或者，如果模型不能满足某些验证，模型的生成者需要将这些说明增加在Model Explanation中。
综上，存在这样一些模型，最后的产生的结果都是一样的，无法用于评分。模型消费者可以选择不去部署这些模型，或者部署他们只为加深理解，而不用于评分。



# REF
[PMML 4.3 - General Structure](http://dmg.org/pmml/v4-3/GeneralStructure.html)