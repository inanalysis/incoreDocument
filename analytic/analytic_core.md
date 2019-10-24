# Analytic Core
## Implement algorithm
要實作一個演算法，有兩個步驟：參數定義 以及 演算法實作
### 參數定義 Parameter defining
演算法的參數定義請存為 算法名.json
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
- **param oject**
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
    
- **input group object**
    此參數說明了此演算法應該要有那些輸入。
    一個算法可以有多個input group，每個input group 允許多個feature，同個 input group 的型態應該要一致。
    - **name**
    - **description**
    - **type**
        此 input 的型態
        `Enum("float","classifiable","string","path")`
    - **amount**
        此輸入可否接受多個 feature
        `Enum("single", "multiple")`
- **output object**
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

    ```
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
                "name": "param2 Name",
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

### 演算法實作 Algorithm Implementing
要實作演算法，請繼承該 projectType 的 base class
以下將先講解parameter object, input group object, output object 的結構
再分別解釋regression, classification, clustering, abnormal 的實作方法
#### Parameter object
#### Input group object
#### Output object