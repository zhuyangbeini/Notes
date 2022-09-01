# 基础语法：

### 变量声明：

#### 全局和局部：

global 变量名  --> 全局，一般声明在代码最最最前面，在rollout之前

local  变量名  -->  局部，如果不写默认是local

函数名大写FN为全局函数，小写fn为局部函数

mat = matrix3 1    --> 生成一个ident的3**4矩阵，其中3* *3是旋转，最后一行是平移

***

### 文件引用：

`filein "UserNormalTranslator.ms"`

需要放在同一目录下

***

### 循环

```maxscript
for i = 1 to 20 do --从1遍历到20，一般用于遍历numfaces（总面数）/numverts（总点数）
```

```maxscript
for i in array do --类似foreach，遍历数组中的元素
```

***

### 判断

```maxscript
if true do ()
```

```maxscript
if true then () else ()
```

或 - or ，且 - and

****

### Debug：

##### 打印到控制台

`print`  自带\n换行符，打印 array 时，会自动遍历打印元素的值

`format "some string % \n" value`  % 与value 成对使用 ，value会自动填充到string中对应的%中



***

# 代码示例：

## 法线相关：

### 通过 Edit_Normal 获取法线

```maxscript
SelMeshOBJEditNormals = theMesh.modifiers[#Edit_Normals].EditNormalsMod
theNumFaces = SelMeshOBJEditNormals.getNumFaces()
for f = 1 to theNumFaces do --loop through the faces in the modifier
(
    for i = 1 to SelMeshOBJEditNormals.getFaceDegree f do --get the vertex number of the face f and loop
            (
                format "----- Current Face Corner Index : % -----\n" i            
                theNormalId = SelMeshOBJEditNormals.GetNormalID f i --get the ID of the vertex normal by index (f,i)
                format "the Normal Index: %\n" theNormalId
                theVertexId = SelMeshOBJEditNormals.GetVertexID f i --get the vertex ID
                format "the Vertex Index: %\n" theVertexId
                theNormal = SelMeshOBJEditNormals.GetNormal theNormalId --get the normal by ID
                format "the Normal: %\n" theNormal
                theVertex = SelMeshOBJEditNormals.GetVertex theVertexId --get the vertex by ID
                format "the Vertex: %\n" theVertex
            )--end i
            print "-----------------------------------------"    
)--end f
```

***

### 计算切线空间

```maxscript
Fn computeTangentSpace selMesh = (

    local theMesh = snapshotAsMesh selMesh
    local tSpace = #()

    -- Do we have to flip faces?
    local flip = false
    local indices = #(1, 2, 3)
    if dot (cross theMesh.transform.row1 theMesh.transform.row2) theMesh.transform.row3 <= 0 do (
        indices[2] = 3
        indices[3] = 2
        flip = true
    )

    for nFace = 1 to theMesh.numFaces do 
    (
        local face = getFace theMesh nFace
        local tface = getTVFace theMesh nFace

        local v1 = getVert theMesh face[indices[1]]
        local v2 = getVert theMesh face[indices[2]]
        local v3 = getVert theMesh face[indices[3]]

        local w1 = getTVert theMesh tface[indices[1]]
        local w2 = getTVert theMesh tface[indices[2]]
        local w3 = getTVert theMesh tface[indices[3]]

        local x1 = v2.x - v1.x
        local x2 = v3.x - v1.x
        local y1 = v2.y - v1.y
        local y2 = v3.y - v1.y
        local z1 = v2.z - v1.z
        local z2 = v3.z - v1.z

        local s1 = w2.x - w1.x
        local s2 = w3.x - w1.x
        local t1 = w2.y - w1.y
        local t2 = w3.y - w1.y

        local r = 1.0 / (s1 * t2 - s2 * t1)

        local tan1 = [(t2 * x1 - t1 * x2) * r, (t2 * y1 - t1 * y2) * r, (t2 * z1 - t1 * z2) * r]
        local tan2 = [(s1 * x2 - s2 * x1) * r, (s1 * y2 - s2 * y1) * r, (s1 * z2 - s2 * z1) * r]

        local normal = normalize (getFaceNormal theMesh nFace)
        if flip do normal = -normal

        local tangent = normalize (tan1 - normal * (dot normal tan1))

        local handedness = if (dot (cross normal tan1) tan2) < 0.0 then -1.0 else 1.0

        local binormal = (normalize (cross normal tangent)) * handedness

        local fCenter = meshOp.getFaceCenter theMesh nFace

        append tSpace (Matrix3 tangent binormal normal fCenter)
    )

    delete theMesh

    return tSpace
)
```

### 法线转顶点色：

```maxscript
Fn ConvertNormal2VertClr selMesh =
(
    select selMesh 
    if (ClassOf selMesh.modifiers[1]) != Edit_Normals then
    (
        addModifier selMesh (Edit_Normals()) 
    )
    SelMeshOBJEditNormals = selMesh.modifiers[#Edit_Normals].EditNormalsMod
    NumVerts = SelMeshOBJEditNormals.GetNumVertices()
    allVertsNorm = #()
    allVertsNormClr = #()
    for i = 1 to NumVerts do
    (
        normal_array = #{}
        selMeshOBJEditNormals.ConvertVertexSelection #{i} normal_array
        normal_result = normal_array as Array
        normID = normal_result[1]
        allVertsNorm[i] =  selMeshOBJEditNormals.Getnormal normID
        tempR = (allVertsNorm[i].x + 1) * 0.5 * 255.0
        tempG = (allVertsNorm[i].y + 1) * 0.5 * 255.0
        tempB = (allVertsNorm[i].z + 1) * 0.5 * 255.0
        tempColor = [tempR,tempG,tempB] as point3
        allVertsNormClr[i] = tempColor
    )
    select selMesh
  --  convertTo selMesh PolyMeshObject                    
    setCommandPanelTaskMode #modify
    subObjectLevel = 1

    --numVerts = (polyop.getNumVerts selMesh) --get the total vertex count
    for j = 1 to NumVerts do 
    (
        selMesh.EditablePoly.SetSelection #Vertex #{j}        --19,GetVertColor.SetVertColor
        theVertColor = selMesh.getVertexColor #VertexColor
        --currentClr = allVertsNormClr[j] as color
        polyop.setvertcolor selMesh 0 j allVertsNormClr[j]--currentClr
    )
    update selMesh()                     
    --convertTo selMesh Mesh                                                         --20,convertTo selMesh Mesh 
    gc()
    messageBox "Use VertexPaint Modifer to Check VertexColor"
)
```

### 顶点色转法线：

```maxscript
Fn ConvertVertClr2Normal selMesh =
(    
    --selMesh = $selection[1]    
    select selMesh
    convertTo selMesh PolyMeshObject
    SetCommandPanelTaskMode #modify
    subObjectLevel = 1
    numVerts = (polyop.getNumVerts selMesh) --get the total vertex count            --21,getPoly,totalVertCount
    finalNorms = #()
--    inverseWorldM = Inverse selMesh.objecttransform                                        
--    CenterPos = centerOBJ.position * inverseWorldM 
    for j = 1 to numVerts do 
    (
        selMesh.EditablePoly.SetSelection #Vertex #{j}
        theVertColor = selMesh.getVertexColor #VertexColor
        currentNorm = theVertColor as point3
        currentNorm.x = (currentNorm.x / 255.0) * 2 - 1
        currentNorm.y = (currentNorm.y / 255.0) * 2 - 1
        currentNorm.z = (currentNorm.z / 255.0) * 2 - 1
        finalNorms[j] = currentNorm
    )
    select selMesh 
    convertTo selMesh Mesh
    addModifier selMesh (Edit_Normals())
    SelMeshOBJEditNormals = selMesh.modifiers[#Edit_Normals].EditNormalsMod
    NumVerts = SelMeshOBJEditNormals.GetNumVertices()
    allVertsNorm = #()
    allVertsNormClr = #()
    for i = 1 to NumVerts do
    (
        normal_array = #{}
        selMeshOBJEditNormals.ConvertVertexSelection #{i} normal_array
        normal_result = normal_array as Array
        normID = normal_result[1]
        allVertsNorm[i] =  selMeshOBJEditNormals.Getnormal normID
        selMeshOBJEditNormals.SetNormal normID finalNorms[i]
        selMesh.Edit_Normals.setnormalexplicit normID 
    )
    update selMesh()
)
```

# UI界面相关：

### 新建脚本窗口

```maxscript
try (DestroyDialog test) catch ()
rollout test "顶点色"
(
    ----code----
)
createDialog test width:200 height:100
----------------------------------------------------------------------------------
rollout UserNormalTranslator  "UserTranslator" width:368 height:656
(
    ----code----
)
normalTranslatorWindow = newRolloutFloater "NormalTranslator" 380 504
addRollout UserNormalTranslator normalTranslatorWindow
```

### 各类button

```maxscript
button 'normBitArr_save' "Save_Group" pos:[120,35] width:75 height:24 align:#left
on normBitArr_save pressed do ()
```

```maxscript
groupBox 'selBox' "SelectOBJ" pos:[9,15] width:300 height:200 align:#left  
label 'normGroup' "NormalGroup :" pos:[20,80] width:75 height:15 align:#left
slider 'Xsld' "" pos:[59,72+200] width:152 height:25 enabled:true range:[-1,1,0] type:#float orient:#horizontal ticks:10 align:#left
editText 'edt3' "" pos:[272,48+200] width:0 height:0 align:#left
```

```maxscript
pickButton 'pickOBJ' "SelectOBJ" pos:[16,35] width:89 height:24 align:#left
on pickOBJ picked obj do ()
```

```maxscript
radioButtons 'rdo2' "" pos:[88,48+200] width:138 height:16 enabled:true labels:#("Add","Multiply") default:1 columns:2 align:#left
on rdo2 changed state do ()
```

```maxscript
checkbox 'abCheckbox' "" pos:[212,152+200] width:16 height:24 checked:false align:#left
on abCheckbox Changed theState do ()
```

```maxscript
slider 'radioSld' "" pos:[56,248+200] width:152 height:25 enabled:true range:[0,1,0] type:#float orient:#horizontal ticks:10 align:#left
on radioSld changed val do ()
```
