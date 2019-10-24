---
layout: default
title: Analytic Core
---

# Analytic Core
<div class="alert alert-dark" role="alert">
    <a href="../../index.html">Document</a>
     > 
    <a href="../index.html">Dev</a>
     > 
    <a href="./index.html">Ananlytic Service</a>
     > 
    Analytic Core
</div>

---
本章節講解如何實作數據分析演算法模組

要實作一個演算法，有兩個步驟：[演算法定義](#演算法定義) 以及 [演算法實作](#演算法實作)

## 演算法定義
演算法的定義請存為 算法名.json
該json中必須定義以下屬性
- **dataType**
`Enum("num","cv","nlp")`
- **projectType** `Enum("classification","regression","abnormal","clustering")`
  每個 dataType 所支持的 projectType

  |   |regression|classification|abnormal|clustering|
  |:--------------:|:------------:|:-------------:|:--------:|:---------:|
  |num|✔|✔|✔|✔|
  |cv|✔|✔|✖|✖|
  |nlp|✔|✔|✖|✖|
- **algoName** 
    實作演算法的名稱 e.g. *R08525000_kmean*。這個名稱應該與檔名一致
- **lib** 
    所使用的函式庫，目前支援 sklean 以及 keras
    `Enum("sklearn","keras")`
- **Parameter oject**
    參數物件有以下這些屬性
    - **name**: "參數名稱"
    - **description**: "此參數的描述"
    - **type**: `Enum("int","float","bool","enum","string")`

        |int|float|bool|enum|string|
        |:-:|:-:|:-:|:-:|:-:|
        |此參數為 int|此參數為 float| 此參數為 bool<br>0 代表 False,1 代表 True|此參數為選項|此參數為string|
    - **default**: 此參數的預設值
        Example:

        |int|float|bool|enum|string|
        |:-:|:-:|:-:|:-:|:-:|
        |10|23.5|1|"option1"|"some string"|
    - **other attribute**
    
        |屬性|意義|required by|example|
        |:-:|:-:|:-:|:-:|
        |upperBound|參數的上界|int, float|20|
        |lowerBound|參數的下界|int, float|0|
        |list|選項參數的候選選項|enum|["op1","op2"]|
    
- **Input group object**
    此參數說明了此演算法應該要有那些輸入組。
    一個算法可以有多個input group，每個input group 允許多個feature，同個 input group 的型態應該要一致。
    - **name**
    - **description**
    - **type**
        此 input 的型態
        `Enum("float","classifiable","string","path")`
    - **amount**
        此輸入可否接受多個 feature
        `Enum("single", "multiple")`
- **Output object**
    此參數說明了算法的輸出。
    一個算法可以有多個output，每個 output 僅能有一個 feature。
    對於沒有輸出的 Unsupervised learning，此物件為一個空list。
    - **name**
    - **description**
    - **type**
        此 output 的型態
        `Enum("float","classifiable","string","path")`
- <details>
    <summary>Example</summary>

    ```json
    {
        "dataType": "num",
        "projectType":"regression",
        "algoname": "yourAlgoName",
        "lib" : "keras",
        "param":[
            {
                "name": "param1Name",
                "description": "param1 Description",
                "type": "int",
                "upperBound": 100,
                "lowerBound": 10,
                "default":20
            },
            {
                "name": "param2Name",
                "description": "param2 Description",
                "type": "float",
                "upperBound": 30.5,
                "lowerBound": 0,
                "default":23.2
            },
            {
                "name": "param3Name",
                "description": "param3 Description",
                "type": "bool",
                "default":1
            },
            {
                "name": "param4Name",
                "description": "param4 Description",
                "type": "enum",
                "list": ["option1","option2","option3"],
                "default":"option2"
            },
            {
                "name": "param5Name",
                "description": "param5 Description",
                "type": "string",
                "default":"default string"
            }
        ],

        "input":[
            {
                "name": "input1Name",
                "description": "input1 description",
                "type": "float",
                "amount": "multiple"
            },
            {
                "name": "input2Name",
                "description": "input2 description",
                "type": "classifiable",
                "amount": "single"
            },
            {
                "name": "input3Name",
                "description": "input3 description",
                "type": "string",
                "amount": "single"
            },
            {
                "name": "input4Name",
                "description": "input4 description",
                "type": "path",
                "amount": "single"
            }
        ],

        "output":[
            {
                "name": "output1Name",
                "description": "output1 description",
                "type": "float"
            },
            {
                "name": "output2Name",
                "description": "output2 description",
                "type": "classifiable"
            },
            {
                "name": "output3Name",
                "description": "output3 description",
                "type": "string"
            },
            {
                "name": "output4Name",
                "description": "output4 description",
                "type": "path"
            }
        ] # For unsupervised project, it is a empty list
    }
    ```
    </details>
    
---

## 演算法實作
要實作演算法，請繼承該 projectType 的 base class

以下將先說明 [parameter object](#parameter-object), [input group object](#input-group-object), [output object](#output-object) 的結構

再分別解釋 [regression](#regression-algorithm), [classification](#classification-algorithm), [clustering](#clustering-algorithm), [abnormal detection](#abnormal-detection-algorithm) 的實作方法

### Parameter object
在程式中，使用者輸入的參數值皆以 dictionary 方式存在 `self.param` 中

例如，如果在參數定義中，有一個 parameter object 為
```json
{
    "name": "param2",
    "description": "param2 Description",
    "type": "float",
    "upperBound": 30.5,
    "lowerBound": 0,
    "default":23.2
}
```
在程式中，此參數的輸入值會存於 `self.param['param2']`

### Input group object

在程式中，使用者對於每個輸入組所指定的數據會以 dictionary 方式存在 `self.inputData` 中

每個輸入組的結構皆為

```python
numpy.array(
    [
        [col1-row1,col2-row1,....,colN-row1],
        ....,
        [col1-rowM,col2-rowM,...,colN-rowM]
    ]
)
```

例如，一個如下的 csv

|a|b|c|
|---|---|---|
|1|2|3|
|4|5|6|
|7|8|9|
|10|11|12|

將 a, b, c 三個 column 以 [b c a] 的順序讀入一個名為 input1 的輸入組，則 `self.inputData['intput1']` 將為

```python
numpy.array(
    [
        [2,3,1],
        [5,6,2],
        [8,9,7],
        [11,12,10]
    ]
)
```
<br>
若該輸入組型態為 *classifiable*，則系統會自動將其轉換為 one-hot encoding 型式，並將對應關係存為 `self.c2d` 及 `self.d2c`

例如，一個如下的 csv

|a|b|c|
|---|---|---|
|cat|apple|red|
|dog|banana|blue|
|dog|orange|black|
|cat|banana|green|

將 a, b, c 三個 column 以 [c,a,b] 順序讀入一個名為 input2 的輸入組，系統會隨機為每個column產生類別對應如下面型式

```python
self.c2d = {
                "a":{
                    "cat":0,
                    "dog":1
                },
                "b":{
                    "apple": 0,
                    "banana": 1,
                    "orange":2
                },
                "c":{
                    "red":0,
                    "blue":1,
                    "black":2,
                    "green":3
                }
           }

self.d2c = {
                "a":{
                    "0":"cat",
                    "1":"dog"
                },
                "b":{
                    "0": "apple",
                    "1": "banana",
                    "2":"orange"
                },
                "c":{
                    "0":"red",
                    "1":"blue",
                    "2":"black",
                    "3":"green"
                }
           }
```

並且 `self.inputData['input2']` 將為

```python
numpy.array(
    [
        [ [1,0,0,0], [1,0], [1,0,0] ],
        [ [0,1,0,0], [0,1], [0,1,0] ],
        [ [0,0,1,0], [0,1], [0,0,1] ],
        [ [0,0,0,1], [1,0], [0,1,0] ]
    ]
)
```

### Output object

### Regression algorithm

### Classification algorithm

### Clustering algorithm

### Abnormal detection algorithm

