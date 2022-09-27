总点数：`i@numpt`运行在points上

总面数：`i@numprim`运行在primitives上

# elem

`i@elemnum`运行在points上时，i@elemnum == @ptnum；

                    运行在primitives上时，i@elemnum == @primnum;

`i@numelem`运行在points上时，i@elemnum == @numpt；

                    运行在primitives上时，i@elemnum == @numprim;

```c
printf("%s\n",value);
```

### 点线面互访

#### 顶点：

顶点一般以面上的第几号来表示，比如`5v3||5:3`表示5号面上的01234..中的3号顶点

顶点生成后会依附在对应的点上，顶点位置随点运动，不能单独控制顶点的位置，操作顶点只在uv模式下可以进行

顶点序号：每个面上的顶点序号，如果是三角面则是0~2，四边面则是0~3

线性顶点序号（linear vertex）：一个立方体，6个四边面，一个面有4个vertex，则线性顶点序号为0~23

```c
int pt = vertexpoint(0,1);//form vertex to point//从顶点访问点 0是输入geo，1是输入‘线性序号’， 返回点序号

int lvt0 = pointvertex(0,7);//point to vertex //从点访问顶点 ，输入点序号，返回第一个顶点的‘线性序号’
int lvt1 = vertexnext(0,lvt0);//输入线性顶点序号，返回住在同一点的下一个顶点的‘线性序号’
int lvt2 = vertexnext(0,lvt1);//再下一个，如果没有下一个，返回-1
int pvt = vertexprev();//返回上一个

setvertexattrib(0,'Cd',-1,lvt0,{1,0,0});
//(geo,'attribName',primnum,vertexIndex,value)
//如果输入指定的面序号，则vertexIndex 只需要输入顶点序号
//如果输入-1为面序号，则vertexIndex 需要输入‘线性顶点序号’


int prim = vertexprim(0,lvt2);//输入‘线性顶点序号’，返回面序号
int vt0 = vertexprimindex(0,lvt2);//输入‘线性顶点序号’，返回该顶点所在面的顶点序号
```

#### 面：

```c
primvertices() //输入primnum，返回这个prim的所有顶点的‘线性序号’,[数组]
primvertex() //输入primnum和顶点序号，返回这个顶点的‘线性序号’
primvertexcount() // 输入primnum,返回这个prim的顶点数量


primpoints() // 输入prinnum,返回这个prim的所有点的序号[数组],有序
primpoint() //输入primnum和顶点序号,返回这个顶点所在的点的序号

```

#### 点：

```c
pointprims() // 输入ptnum，返回包含这个点的prim的序号[数组]
pointvertices() // 输入ptnum,返回住在这个点上的所有顶点的‘线性序号’[数组]
pointvertex() //输入ptnum,返回住在这个点上的‘第一个’顶点的‘线性序号’
```

##### 查找点邻居（手写版）

```c
//i[]@nbs = neighbours(0,@ptnum); //系统自带函数
int nbprims[] = pointprims(0,@ptnum);//获得点所在的面序号
int primverts[];
int allpts[];
foreach(int prim;nbprims)//遍历每个面
{
    primverts = primvertices(0,prim);//获取面上顶点的线性序号
    foreach(int vertid;primverts)//遍历顶点序号
    {
        int pt = vertexpoint(0,vertid);//根据线性序号获得点序号
        append(allpts,pt);//存储每个面的点序号，这里面重复两次的，即为该点邻居
    }
}

//遍历数组找出重复两次的点序号
int nbs[];
foreach(int pt;allpts)
{
    int count = 0;
    for(int i =0;i<len(allpts);i++)
    {
        if(pt == allpts[i])
        {
            count++;
        }

    }
    if(count == 2)
    {
    //每个面都会求出两个点与之相邻，所以会有6个值，两两重复，要判断一下是否已经存在在数组中了
    //find(),如果在指定数组中找到该值，则返回该值在数组中的index，找不到返回-1
        if(find(nbs,pt)<0)
           append(nbs,pt);
    }
}
i[]@nbs = nbs;
```

### Function

```c
function int myfun(const int a,b)
{
    return 0;
}
```

<mark>！！！</mark>Houdini中的函数是引用传递，不是值传递，用const修饰只读不可写，可防止意外修改了原值
