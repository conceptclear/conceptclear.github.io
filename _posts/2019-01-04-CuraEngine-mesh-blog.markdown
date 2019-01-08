---
layout: post
title:  "CuraEngine开源程序解读————面片读取"
date:   2019-01-04 10:35:56
category: 3dprint
---

## mesh                 
mesh中主要定义了关于三角形网格三个类和这三个类的一些成员函数，分别是MeshVertex，MeshFace以及Mesh。           

### 全局变量             
`const int`类型的全局变量vertex_meld_distance，其值设定为MM2INT(0.03)；          
### 全局函数                
- **static inline uint32_t pointHash(const Point3& p)**。构造了一个基于vertex_meld_distance的函数，所有在vertex_meld_distance范围内的点都会映射到一个哈希值上。           


### MeshVertex                  
MeshVertex类是在网格中使用的顶点类型，其包含了顶点的坐标信息以及跟踪连接到这个点上的面片信息。             
#### 成员变量                  
`Point3`类型的成员变量p，用于存储顶点的坐标信息；            
`uint32_t`类型的vector容器connected_faces，用于存储连接面的索引列表。                
#### 成员函数               
- **构造函数**。构造函数中只提供Point3类型的参数p用于初始化成员变量p，默认给connected_faces容器预留8个空间大小；         

### MeshFace                
MehsFace类用来表示一个三维模型中的一个三角形面片，面片包含了三个顶点的信息以及三条边所连接的其它三个面片的信息。一个正确的模型同样有可能一条边连接了超过两个面，在这种情况之下，存储在connected_face_index数组中的面片为连接模型外部的面片。        
#### 成员变量                  
`int`类型的vertex_index[3]数组，用于存储三个顶点的索引，三个顶点按照逆时针排序；         
`int`类型的connected_face_index[3]数组，用于存储连接三个边的另一个面片的索引号，其中connected_face_index[0]对应的面片和当前面片所共享的边是vertex_index[0]和vertex_index[1]所连接形成的边。       

### Mesh                  
Mesh类是3D模型最基本的表示类，以MeshFace存储所有面的信息。Mesh类是SettingsBase类的继承类，SettingsBase是设置一些值的基础类，这里可以不做深究。             
#### 成员变量                 
`<uint32_t, std::vector<uint32_t> > `类型的unordered_map容器vertex_hash_map，用于存储该位置的散列的每个顶点的索引引用，允许快速检索具有相同位置的点。类型的unordered_map容器相较于map容器具有更高的查询效率，采用以哈希表为底层；            
`AABB`类型的成员变量aabb，用于构建整个模型的包围盒；                
`MeshVertex`类型的vector容器vertices，用于存储网格的所有顶点数据；              
`MeshFace`类型的vector容器faces，用于存储网格的所有面片数据；            
`bool`类型的has_disconnected_faces，判定mesh中是否存在断开连接的面；                 
`bool`类型的has_overlapping_faces，判定mesh中是否存在交叠的面；                 
`Settings`类的对象settings，用于存储基础设定；                  
`string`类型的mesh_name，用于存储读取进来的模型名称。                   
#### 成员函数             
- **构造函数**。构造函数中不直接输入模型的面片信息，而是对其一些设定值进行初始化，形参为其虚基类SettingsBaseVirtual类所构造对象的指针；                 
- **int findIndexOfVertex(const Point3& v)**。私有成员函数，用于返回接近该Point的顶点索引，若不存在则新建一个索引并返回；寻找索引时，通过所搜寻点的hash值在vertex_hash_map中寻找，若寻找到同样hash值的存储点，通过判断寻找到的点和所搜寻点之间的距离是否超过设定的全局变量vertex_meld_distance来判断该点是否满足条件，若满足则返回所搜寻点的索引值，若不满足则在vertex_hash_map[hash]中添加一个当前顶点数量的值，在vertices容器中添加该点，并返回vertices.size() - 1；                 
- **void addFace(Point3& v0, Point3& v1, Point3& v2)**。用于向faces容器中添加面片。首先检测三个点的索引值是否有两个指向同一个值上，若指向同一个值则直接返回；若是一个新的面片，则向faces容器中直接向后添加一个成员，并给这个成员的vertex_index赋值，同时分别对这三个顶点的connected_faces添加该面片索引；                        
- **void clear()**。用于清除所有数据（清空faces，vertices和vertex_hash_map）；                        
- **void finish()**。用于完成模型connected_faces的设置。完成网格输入之后，首先清除vertex_hash_map容器，因为这个容器不再是必须的而且占据了很大的一部分内存空间。在addFace中已经确定了每个顶点所连接的面片，这里再通过getFaceIdxWithPoints函数确定每个面片上每条边所连接的另一个面片的索引号；                         
- **Point3 min() const**。用于返回AABB包围盒最小点；                         
- **Point3 max() const**。用于返回AABB包围盒最大点；                         
- **AABB3D getAABB() const**。用于返回AABB包围盒；                         
- **void expandXY(int64_t offset)**。用于拓展AABB包围盒。若offset为正，向外拓展offset；若为负，向内拓展；                         
- **void offset(Point3 offset)**。用于平移整个模型，包括aabb和所有的顶点；                         
- **int getFaceIdxWithPoints(int idx0, int idx1, int notFaceIdx, int notFaceVertexIdx) const**。用于根据当前面片索引以及对应边的顶点索引来确定所连接的另一个面片的索引，当多个面连接相同的边缘时，若连接一条边的面片数量为单数，则说明存在断开连接的面，若为双数则从idx1到idx0查看，返回下一个顶点构成的是一个逆时针面的面片；                         

## MeshGroup
MeshGroup中只定义了一个MeshGroup类，主要用于作为保存一个或者多个mesh。一个MeshGroup中保存的模型都是在一次打印中需要被打印的模型，所以在同一次打印中，只会有一个MeshGroup。          

### MeshGroup类
MeshGroup类是NoCopy类的继承类，所以不能直接进行拷贝。          
#### 成员变量
`Mesh`类型的vector容器meshes，用于存储多个mesh模型；           
`Settings`类的对象settings，用于基础设定。                         
#### 成员函数
- **Point3 min() const**。用于返回AABB包围盒最小点；                         
- **Point3 max() const**。用于返回AABB包围盒最大点；                        
- **void clear()**。对meshes中每个mesh都进行clear；                  
- **void finalize()**。主要是用于调整mesh的位置。                     
#### 全局函数
- **bool loadMeshIntoMeshGroup(MeshGroup* meshgroup, const char* filename, const FMatrix3x3& transformation, Settings& object_parent_settings)**。从文件中读取网格并将其存储在MeshGroup的meshes当中，其中meshgroup为网格存储位置，filename为文件名，transformation为对该模型所有顶点的变换矩阵，object_parent_settings为父类设置，函数返回是否成功读取面片模型并将其保存于meshgroup当中。该函数通过调用`loadMeshSTL_ascii`或`loadMeshSTL_binary`对ASCII格式或二进制格式的STL文件进行读取。                 
