# 说明
* 基于PMML Version4.3
* 基本可以视为官方文档的一个翻译
* 如有疏漏，请联系yao544303963@gmail.com

# 数据挖掘模式
数据挖掘模式，可以看做是模型的一个看门人。所有进入模型的数据，必须经过数据挖掘模式。每个模型都包含且只包含一个数据挖掘模式，用于列出该模型使用的数据。数据挖掘模式包含针对特定模型不同的信息，相对的，数据字典中定义则是稳定的，不会随模型变化而变化。数据挖掘模式的主要目的是列出使用模型需要的数据。

数据挖掘字段也定义了每个字段的使用用途（激活、追加、目标）以及针对空值、非法数据的策略。

# Schema
```
<xs:element name="MiningSchema">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
      <xs:element maxOccurs="unbounded" ref="MiningField"/>
    </xs:sequence>
  </xs:complexType>
</xs:element>

<xs:element name="MiningField">
  <xs:complexType>
    <xs:sequence>
      <xs:element ref="Extension" minOccurs="0" maxOccurs="unbounded"/>
    </xs:sequence>
    <xs:attribute name="name" type="FIELD-NAME" use="required"/>
    <xs:attribute name="usageType" type="FIELD-USAGE-TYPE" default="active"/>
    <xs:attribute name="optype" type="OPTYPE"/>
    <xs:attribute name="importance" type="PROB-NUMBER"/>
    <xs:attribute name="outliers" type="OUTLIER-TREATMENT-METHOD" default="asIs"/>
    <xs:attribute name="lowValue" type="NUMBER"/>
    <xs:attribute name="highValue" type="NUMBER"/>
    <xs:attribute name="missingValueReplacement" type="xs:string"/>
    <xs:attribute name="missingValueTreatment" type="MISSING-VALUE-TREATMENT-METHOD"/>
    <xs:attribute name="invalidValueTreatment" type="INVALID-VALUE-TREATMENT-METHOD" default="returnInvalid"/>
  </xs:complexType>
</xs:element>

<xs:simpleType name="FIELD-USAGE-TYPE">
  <xs:restriction base="xs:string">
    <xs:enumeration value="active"/>
    <xs:enumeration value="predicted"/>
    <xs:enumeration value="target"/>
    <xs:enumeration value="supplementary"/>
    <xs:enumeration value="group"/>
    <xs:enumeration value="order"/>
    <xs:enumeration value="frequencyWeight"/>
    <xs:enumeration value="analysisWeight"/>
  </xs:restriction>
</xs:simpleType>

<xs:simpleType name="OUTLIER-TREATMENT-METHOD">
  <xs:restriction base="xs:string">
    <xs:enumeration value="asIs"/>
    <xs:enumeration value="asMissingValues"/>
    <xs:enumeration value="asExtremeValues"/>
  </xs:restriction>
</xs:simpleType>

<xs:simpleType name="MISSING-VALUE-TREATMENT-METHOD">
  <xs:restriction base="xs:string">
    <xs:enumeration value="asIs"/>
    <xs:enumeration value="asMean"/>
    <xs:enumeration value="asMode"/>
    <xs:enumeration value="asMedian"/>
    <xs:enumeration value="asValue"/>
  </xs:restriction>
</xs:simpleType>

<xs:simpleType name="INVALID-VALUE-TREATMENT-METHOD">
  <xs:restriction base="xs:string">
    <xs:enumeration value="returnInvalid"/>
    <xs:enumeration value="asIs"/>
    <xs:enumeration value="asMissing"/>
  </xs:restriction>
</xs:simpleType>
```

# 说明
## MiningSchema
数据挖掘模式的顶层元素，子元素有Extension 和 MiningField，可重复出现。

## MiningField
数据挖掘模式中的字段。

**包含属性：**  
**name**:
字段的标识，必须和数据挖掘模式父元素中的字段定义域保持一致（即必须和数据字典中的字段定义保持一致）  
关于字段的定义域，详见[Scope of Fields](http://dmg.org/pmml/v4-3/FieldScope.html)
如果数据字典为某个字段定义了一个**displayName**, **name**仍是可以识别的，**displayName**用在接口处，方便用户理解，在模型内部流转还是使用**name**。

**usageType**:
描述该字段的用途，有如下可选值：
* active：模型输入字段
* target：有监督模型的目标值
* predicted：模型预测值。在4.2版本中，不建议这个用法，被target替代
* supplementary：额外描述属性。在模型中，额外描述用于解释说明，但不是必须的。当模型中需要一些预处理的数据转化（如标准化），这时，追加一个额外描述属性，用来描述原始值的情况。
* group：类似于SQL中的group by。举个例子，这个经常在关联规则和序列模型中使用，将物品根据用户ID或者交易ID分组。
* order：类似于SQL中的order by。根据物品编号或者交易编号进行排序。目前用在序列模型和时序模型中。
* frequencyWeight 和 analysisWeight：这些字段对于预测结果的评分没什么帮助，但提供了模型构建时非常重要的信息。  
frqquencyWeight 通常是一个正整数，有时也被称为"replication weight"。它的值表示每条记录在数据集中出现的次数。
analysisWeight 是一个正小数，通常用来描述回归模型的回归系数和树模型的条件权重。它的值表示模型中各个情况的重要性。
  
数据挖掘模式中的对于target 字段的定义不是必须的。在大多数情况下，该字段的定义对最后预测结果没有影响。对于有监督模型，提供了target值的相关信息，而该值则有模型自行计算，故定义不是必须。然而，在如下情况下，必须对target 进行定义：
* 模型拥有超过一个target 字段，这时解释说明就是必须的了，比如KNN
* 模型需要计算残差作为一路输出

**optype**:
该属性值用于重载DataField中相关属性值。这意味着，一个DataField可以在不同的模型中，实现不同的操作。比如，1可以在回归模型中当成数值型输入，也可以在数模型中作为类型输入。

**importance**:
表名该字段的重要性。这个属性用于在预测模型中表述每个特征对最终结果的重要程度，类似RF或者XGBoost 输出的importance。

**outliers**:
异常值处理，有以下三种处理方式
* asIs：按照原始值处理
* asMissingValues：当成缺失值处理
* asExtremeValues：转化为MiningField中的区间边界值。

**highValue 和 lowValue**:
outliers 中asExtremeValues 处理方式时的边界值
* if x < lowValue then x = lowValue
* if x > highValue then x = highValue
注意，异常值处理支队定义在该数据挖掘模式中字段有效，对原始字段（即定义在数据字典中的）无效。



在数据字典对 values 元素的描述中，有三个属性：valid（有效），invalid（无效）和missing（缺失）。针对无效和缺失的值，有如下属性相对应。

**missingValueReplacement**:
该属性有值时，遇到输入缺失时，用该值替代缺失。前提是模型支持这种做法。  

**missingValueTreatment**: 
该属性是**missingValueReplacement**的一个说明。  

**invalidValueTreatment**:
该属性用于指定，当输入无效时的处理策略。
* returnInvalid： 模型返回一个值，表名输入无效
* asIs： 按照原始值处理
* asMissing：当成缺失值处理



# REF
[PMML 4.3 - Mining Schema](http://dmg.org/pmml/v4-3/MiningSchema.html)