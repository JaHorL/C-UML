# C++编程建议与UML文档描述

本文主要讲解两块内容，一块主要是想针对编程中遇到的问题给出一些建议和优化思路，另一块是想介绍一下UML模型，帮助大家梳理自己负责的代码。

## 类与类之间的关系

类是代码中的重要组成元素，它把数据和方法当成一个整体来对待。它的很大一个作用就是限定一个方法和数据的职责。举个例子，我们目标检测的后处理是需要**把模型输出的feature-map解析成一个个障碍物**，完成这个功能需要用到很多参数和数据，但这些数据是给一个目标服务的。那么我们就可以用类来实现这个功能，把这个功能当做整体工程中的一个细胞。

### 实现的建议

我们现在知道类的功能是信息封装和限定职责，那么怎么去设计一个类比较好呢？

我给一些小建议：

1. **单一职责**，这一条是必须的。假如我设计一个类，这个类的作用包含tracking和detection，那明显是不合理的。单一职责有几个好处：
   * 有效限制各个类的职责，加快问题的定位与程序修改的范围。
   * 降低复杂度，增加可维护性，可读性。
2. **尽量隐藏更多信息**，如把数据信息和一些不会被外部使用的函数隐藏在类的内部。
   * 类的使用者经常不是自己（或者是很久之后的自己），如果我们提供更少和更明确的信息，那么使用者能够最快速得掌握类的使用方法。
   * 如果使用者有随意修改参数的权限，可能会出现不可预计的错误。

### 优化与关系的描述

#### part 1

##### 代码整理和优化

我们现在有一个最原始版的detector类，这个类实现的质量不错也很稳定，但是我有点吹毛求疵，提了两个小小的建议：

1. 我使用Initialize做初始化会不会更好呢？ 
2. 如果我把类中的需要初始化的参数整理成一个数据结构，那么我的Init需要传入的参数就很少，调用起来多方便。

```c++
class Detector {
public:
  Detector(int input_image_width, int input_image_height, int num_classes,
           std::vector<float> confidence_threshold_arry, float nms_threshold,
           bool is_tensorflow);
  ~ Detector();
  std::vector<std::vector<float>> Detect(std::vector<float *> reslut_blobs,
                                         NmsMode nms_mode);
protected:    
    
private:
  // nms parameters
  int num_classes_;
  int num_anchors_;
  int num_stages_;
  int num_total_anchors_;
  int coords_;
  int c_num_;
  float confidence_threshold_;
  std::vector<float> confidence_threshold_arry_; //[MAX_CLASS_NUM] = {0.0};
  float objectness_threshold_;
  float nms_threshold_;
  int input_image_width_;
  int input_image_height_;
  std::vector<float> anchors_;
  std::vector<int> mask_;
  std::vector<int> anchors_scale_;
  bool is_tensorflow_;
  void GetRegionBox(std::vector<float> &pred_bbox, const float *pred,
                    const std::vector<float> &anchors, int n, int index, int i,
                    int j, int w, int h, int num_hw);
};

```

稍稍整理后：

```c++
struct DetectorParams {
    int num_stages = 3;
    int input_image_width = 384;
    int input_image_height = 640;
    
    int num_anchors;
    int num_classes;
    int offset_objectness;
    float objectness_threhold;
    int offset_box_objectness;
    int coords;
    float nms_threshold; 
    int one_anchor_c_num;
    int c_num;
    std::vector<float> anchors;
    std::vector<int> mask;
    std::vector<int> anchors_scale;
    std::vector<float> confidence_threshold_array;
    NmsMode nms_mode;
};


class Detector {
public:
  Detector();
  ~ Detector();
  void Initialize(const DetectorParams &params);
  std::vector<std::vector<float>> Detect(std::vector<float *> reslut_blobs);
protected:    
    
private:
  // nms parameters
  DetectorParams params_;
  void GetRegionBox(std::vector<float> &pred_bbox, const float *pred,
                    const std::vector<float> &anchors, int n, int index, int i,
                    int j, int w, int h, int num_hw);
};

```

##### UML文档描述

单个类的基本元素包含**方法和属性**，其中根据**可见性**，可以分成**公有、私有和受保护**这三类。

因此UML中类的基本描述包含：

| 区域 | 属性                      |
| :--- | :------------------------ |
| 1区  | 类名 (还可以包含构造型)   |
| 2区  | 类的属性区 (内部数据元素) |
| 3区  | 行为 - 私有的和公共的     |

可见性的标注如下：                

| 标记 | 含义                                     |
| ---- | ---------------------------------------- |
| -    | 私有的，说明对类外部的调用者是不可见的。 |
| #    | 保护类型的，只对子类是可见的             |
| +    | 公共的，对所有都是可见的                 |

我们可以用UML图来表示Detector和DetectorParams这**两个类**， 这里的两个类是**强聚合**的关系，因此用**强聚合**符号来描述：

![PlantUML diagram](http://www.plantuml.com/plantuml/png/TL1HI_D047pk_Og13qhRhmq-1mbL1Ec38le3mfcSoMBkBkGsLHJzxUQkYQIDVKcOdTsPkTawbWstkXBkbKmj6wcHLTAvA-Q6dbjMkyoQ9_fGk121m8rYbR5jy2c0__WUy68Py0dS6MAILIUmw8GSz_o3TtAOBn5ZRod7w7RXE8_ZVL2vVAciPAoIDUwrkxEvyXVZXSboaWZWdnVSe0usAUBVM0VZGDugVbG5MNDjtwUU2UPh72Bt6GRuzoePNdlJsMm6006V-mol5dxnInmLEqGCzVfIJAjJnx9G3h4_xiTawfPJqXsHlb7EvLcsL5IKFAQ3fZHPQHitcOSmHsPH54t-NUeXBrQRpMc_MsNoLbS_)

**语法：**

```shell
@startuml
skinparam classAttributeIconSize 0
class Detector {
  + ~Detector()
  + Detector()
  + Initialize(const DetectParams &) : void 
  + Detect(std::vector<float *>, NmsMode) : std::vector<std::vector<float>>
  # 
  - params_ : DetectorParams
  - GetRegionBox(std::vector<float> &, const float *, const std::vector<float> &, int, 
  			     int, int, int, int, int, int) : void
    }

class DetectorParams {
    + num_stages : int
    + input_image_width : int
    + input_image_height : int
    + num_anchors : int
    + num_classes : int
    ...
    ...
    ...
}

Detector *-- DetectorParams
@enduml
```

#### part2

##### 代码整理与优化

之前我们只有**DetectTypeA**一种检测方法，而现在我们又增加**DetectTypeB**的检测方法，那我们如何去**扩展我们的detector**更合适呢？

首先提出有一种简单粗暴的方法，我啥都不考虑**怼接口**就完事了。

```c++
struct DetectorParams {}
class Detector {
public:
  Detector();
  ~ Detector();
  void Initialize(const DetectorParams &params);
  std::vector<std::vector<float>> DetectType1(std::vector<float *> reslut_blobs);

  std::vector<std::vector<float>> DetectType2(std::vector<float *> reslut_blobs);
protected:    
    
private:
  // nms parameters
  DetectorParams params_;
  void GetRegionBoxType1(std::vector<float> &pred_bbox, const float *pred,
                      const std::vector<float> &anchors, int n, int index, int i,
                      int j, int w, int h, int num_hw);
  void GetRegionBoxType2(std::vector<float> &pred_bbox, const float *pred,
                      const std::vector<float> &anchors, int n, int index, int i,
                      int j, int w, int h, int num_hw);

};
```

但是这里两个问题：

1. DetectType1与DetectType2的输出的都是std::vector < std::vector<float>>，但是vector包含的信息是不同的，那么我就需要**两个解析vector的函数**，会增加了**代码的复杂度**。
2. DetectType1与DetectType2 中**95%的内容是重复的**，这会增加了代码的**冗余性**，不好维护。

我们先来解决**第一个问题,** 有没有一个通用的数据结构来表示DetectType1与DetectType2的输出呢？有！

我们可以把函数的输出设置为一个struct，struct可以作为一个通用的数据传输格式，非常好解析，也非常好理解

```c++
struct Obstacle {
  int cutin_type;
  int cls;
  float score;
  float x;
  float y;
  float w;
  float h;
  float dist;
  std::vector<Eigen::Vector2f> box3d;
  std::vector<Eigen::Vector2f> box2d;
};

typedef std::vector<Obstacle>  ObstacleList;

ObstacleList Detector::DetectType1(std::vector<float *> reslut_blobs);
ObstacleList Detector::DetectType2(std::vector<float *> reslut_blobs);
```

我们来解决**第二个问题**，有没有办法来最大化来提高代码的复用性呢?  有！

首先我们可以子类继承父类的方法复用相同的代码与函数，然后通过动态绑定的方法调用不能被复用的代码。 经过分析之后发现，DetectTypeA和DetectTypeB中只有GetRegion函数不一样，那么我们Detect函数就可以被完全复用了。我们只需要在子类中实现各自的GetRegion函数就可以了。

```c++
class BaseDetector {
  public:
    BaseDetector();
    virtual ~BaseDetector();

    void Initialize(const DetectorParams &params);
    ObstacleList Detect(const std::vector<float *> &result_blobs);
    std::vector<int> GetOutputSizes(const DetectorParams &params);
  protected:
    int GetSumOutputSize(const DetectorParams &params);

  private:
    DetectorParams params_;
    virtual BBox GetRegionBox(const float *pred, const std::vector<float> &anchors,
                              int n, int index, int i, int j, int w, int h, 
                              int num_hw,int offset_cls_objectness) = 0;
};


class DetectTypeBDetector : public BaseDetector {
	public:
		DetectTypeBDetector();
		~DetectTypeBDetector();
    private:
		BBox GetRegionBox(const float *pred, const std::vector<float> &anchors,
						  int n, int index, int i, int j, int w, int h, int num_hw,
									  	int offset_cls_objectness) override;
};


class DetectTypeADetector : public BaseDetector {
	public:
		DetectTypeADetector();
		~DetectTypeADetector();
  private:
		BBox GetRegionBox(const float *pred, const std::vector<float> &anchors, 
                          int n, int index, int i, int j, int w, int h, int num_hw,
					      int offset_cls_objectness) override;
};
```

##### UML文档描述

这里类与类之间就又多了一种新的关系----继承， 我们用继承符号描述父类与子类的关系。

![PlantUML diagram](http://www.plantuml.com/plantuml/png/tLFTojf04Bt-zYa62gAjWLwBY0c58XGijVTXJSQwThCRTcRL_jJ7wWFqoRh9ffIjfW_m2MQICoTppfmaQsBfs3TQq6TbMkba0vMMH3cpKtlFkAcisQcl2Az5tu124hv1negjWsy2NYN8TDOXjtjeOnGhgSEaQPX83B7zlfCacCnW0MUhQWZSeJNmYEl5ujnFMXwqbGmlLz3HjtjYMMaitobY0I_WBVBEDrlFhUVTlqBM3LA_VZRKtQuC2yYWxT4o02alzBg17_2JigQmr-cTZnLCvX0b1bz_BXzJW-0S1C0Jn5y4HWvz7ignbxhFHVmGCV_1viSlBHR9OxZ7O_AtaKQMR9ViqJgi_xUBDDNHEfgBfx-RWEbzXt-Dr-4qbbCibrYEiHZVbCTBEi4U3eHSLffAkpy5IOD4Cx1dT4xL618fWjL7IUGZIUHZIUJF98daOrn-px97hySOAqr2hD7Klj6_0G00)

语法：

```shell
@startuml
skinparam classAttributeIconSize 0
class BaseDetector {
 + BaseDetector()
 + ~BaseDetector()
 + Initialize(const DetectParams &) : void 
 + Detect(const std::vector<float *> ) : Obstacle_List
 # GetSumOutputSize(const DetectorParams ¶ms) : int
 - params_ : DetectorParams
 - GetRegionBox(std::vector<float> &, const float *, const std::vector<float> &, int, 
  	        int, int, int, int, int, int) : virtual void
}

class DetectTypeBDetector {
 +DetectTypeBDetector()
 +~DetectTypeBDetector()
 -GetRegionBox(const float *pred, const std::vector<float> &anchors,
	       int n, int index, int i, int j, int w, int h, int num_hw,
	       int offset_cls_objectness) override : BBox
}

class DetectTypeADetector {
 +DetectTypeADetector()
 +~DetectTypeADetector()
 -GetRegionBox(const float *pred, const std::vector<float> &anchors,
	       int n, int index, int i, int j, int w, int h, int num_hw,
	       int offset_cls_objectness) override : BBox
}
BaseDetector <|-- DetectTypeBDetector
BaseDetector <|-- DetectTypeADetector
@enduml
```

#### part 3

##### 代码整理和优化

我们已经把detector这个模块的功能实现得差不多了，需要知道以下三件事才能使用这个detector模块：

1. 如何去实例化
2. 如何去初始化
3. 如何去调用功能接口

但是现在我使用它需要掌握的信息还是太多了，使用方法能不能更加简单一点呢？OK！我们使用使用factory设计模式去控制类的实例化和实例化。

```c++
enum struct DetectorType {DetectTypeA, DetectTypeB}; 

class DetectorFactory {
  public:
    DetectorFactory();  
    ~DetectorFactory();
    std::shared_ptr<BaseDetector> CreateDetector(DetectorType option);
  private:

};

DetectorFactory::DetectorFactory() {}
DetectorFactory::~DetectorFactory() {}

std::shared_ptr<BaseDetector> DetectorFactory::CreateDetector(DetectorType option) {
  std::shared_ptr<BaseDetector> detector = nullptr;
  switch (option) {
    case DetectorType::DetectTypeA: {
      detector = std::make_shared<DetectTypeADetector>();
      DetectorParams params= GetDetectTypeADetectorParams();
      detector->Initialize(params);
      break;
    }
    case DetectorType::DetectTypeB: {
      detector = std::make_shared<DetectTypeBDetector>();
      DetectorParams params= GetDetectTypeBDetectorParams();
      detector->Initialize(params);
      break;
    }
    default: {
      assert(0);
    }
  }
  return detector;
}
```

##### UML文档描述

我在这里用UML完整得描述一下detector模块，其实在工程中类与类之间95%关系，强聚合（组合）和继承（实现）。

![PlantUML diagram](http://www.plantuml.com/plantuml/png/bP1DIi0m48NtESMi2mKFuA8D8hXIq0kaSHR5_9Ha0jRQkslyIMES2cvAcM_clPStH5A1aZKYFirkKK5Pq4R5E1A5UKg4Dzgx-_a5uK9y090guXKIQl81jlrh-ZbvM1SSlo73v0dpuIvRnqFlTegajC5Z8gL_Xbztrof_rmpG9Gi5P3lOH9Mh-fTY5qnYwFg-ILGV_RLMMk7vgLj-5UjHOAjSglb9Bb_V4IF4RxLPHDjdnwYnU__GgvE80TrZikOD)

```
@startuml
skinparam classAttributeIconSize 0
class BaseDetector {}
class DetectTypeADetector{}
class DetectTypeBDetector{}
class DetectorParams {}
class Obstacle {}
class BBox {}
class DetectorFactory {}
enum  DetectorType {
DetectTypeA
DetectTypeB
}
DetectTypeADetector *-- DetectorParams
DetectTypeBDetector *-- DetectorParams
BaseDetector *-- DetectorParams
DetectTypeADetector *-- BBox
DetectTypeBDetector *-- BBox
BaseDetector *-- Obstacle
BaseDetector <|-- DetectTypeBDetector
BaseDetector <|-- DetectTypeADetector
DetectorFactory *-- DetectTypeBDetector
DetectorFactory *-- DetectTypeADetector
DetectorFactory *-- BaseDetector
@enduml
```



## 模块与模块之间的关系

到这里我们的detector模块已经比较完整了，那我们如何去描述模块和模块之间的关系？如何去设计好一个模块呢？

### 实现的建议：

一般情况下，每个子文件夹都中内容都包含一个模块，我针对模块的实现也给出小建议。

1. 模块之间的代码尽量不要出现耦合的情况（common模块和data模块除外），模块应该各司其职，一个模块完成独立的一个功能。模块之前的交互只应该是输入与输出的数据。

   好处：

   * 有效限制各个模块的职责，加快问题的定位与程序修改的范围。
   * 可读性， 可维护性。

2. 单一职责，一个模块只执行模块名表述的职责。

   * 可读性， 可维护性。

### 优化与关系描述

#### part 1

##### 整理与优化

原文件目录的结构：

```
|-- src // 代码
    |-- detection
   		|-- detect_utils.h
   		|-- detect_utils.cpp
   	|-- perception 
   		|-- front_obstacle_perception.h
   		|-- front_obstacle_perception.cpp	
...
...
...
```

整理代码之后，工程的文件结构如下，我在这里做了一些细微的优化。

1. 建立一个common模块，将一些在不同模块间传输的数据结构放入data_type.h中，将对象获取初始化参数的模块放在params.cpp中，之后统一使用params.cpp来获取工程初始化类的参数。
2. 将detector模块中通用的函数放入detector/common.cpp中。

          |-- src // 代码
          	|-- common
          	 	|-- data_type.h
          	 	|-- params.cpp
          	 	|-- params.h
              |-- detector
             		|-- base_detector.h
             		|-- base_detector.cpp
             		|-- common.h
             		|-- common.cpp
             		|-- detector_factory.cpp
             		|-- detector_factory.h
             		|-- DetectTypeB_detector.h
             		|-- DetectTypeB_detector.cpp
             		|-- DetectTypeA_detector.h
             		|-- DetectTypeA_detector.cpp
             	|-- perception 
             		|-- front_obstacle_perception.h
             		|-- front_obstacle_perception.cpp	
          ...
          ...
          ...
##### UML文档描述

detector与common模块和perception模块有交互，detector \ common \ perception摸块在UML中用包（pakage）来表示。

包之间的关系如何描述：

![PlantUML diagram](http://www.plantuml.com/plantuml/png/bLBDIiD04BxFK-nPAFW07jeW1KyMz0Mcsq52ip_iJi2gUNSTqYHro2xqqiFtFzjiCsFYtdMGpwsFcD0Pss7EE-RK7dkc5nlyM_j5vX4WeZtZ1vb8oLBaDdZp3MOqc7qAdb-FcT5sTBXH330irXCnMGxfppZQ6ipqF8F35HsHzqkatKIkS4s12scFydkPyQO9dsg93Sx9FEKyo1laPSbqaI3aUpSBPS0rScMjUTXiL2ReuGMPl4YDBxN9ZQf3aJfvFku_y_GpbkgWcchP0ke_aABLjBgkddUx14fVtbr2rLBLJZ1ioPzwr_q2)

```
@startuml
skinparam classAttributeIconSize 0

package perception {
  class FrontObstaclePerception {}
}

package detector {
class BaseDetector {}
class DetectTypeADetector{}
class DetectTypeBDetector{}
class DetectorFactory{}
class BBox {}
}

package common {
class DetectorParams {}
class Obstacle {}
enum DetectorType {}
}

DetectTypeADetector *-- DetectorParams
DetectTypeBDetector *-- DetectorParams
BaseDetector *-- DetectorParams
DetectTypeADetector *-- BBox
DetectTypeBDetector *-- BBox
BaseDetector *-- Obstacle
BaseDetector <|-- DetectTypeBDetector
BaseDetector <|-- DetectTypeADetector
DetectorFactory *-- DetectTypeBDetector
DetectorFactory *-- DetectTypeADetector
DetectorFactory *-- BaseDetector
FrontObstaclePerception *-- BaseDetector
FrontObstaclePerception *-- DetectorFactory
FrontObstaclePerception *-- Obstacle
@enduml
```

#### part 2

##### UML文档描述

如果我把包 \ 类的关系中所有元素都描述显示在同一个视图中。那么这个视图中包含的内容会非常多，这

反而会变得不清晰，分散使用者注意力。我在这边提一个建议，可以这么做：

1. 对一个模块内部进行详细描述，把模块内部的所有信息描述在一个视图中。
2. 对于模块与模块之间的关系可以简约一些信息，如类中的元素 \ 方法。
3. 对于每个模块我们都展现两个视图：模块内部信息的详细视图  \ 模块与模块关系的简约视图。

## 行为描述

使用时序描述和活动描述即可比较清晰得描述我们系统。

### 时序图(Sequence)描述

时序图(Sequence Diagram)，又名序列图、循序图，是一种UML交互图。它通过描述对象之间发送消息的时间顺序显示多个对象之间的动态协作。

#### 时序图的元素

我们在画时序图时会涉及7种元素：**角色(Actor)、对象(Object)、生命线(LifeLine)、控制焦点(Activation)、消息(Message)、自关联消息**、组合片段。其中前6种是比较常用和重要的元素，剩余的一种组合片段元素不是很常用，也比较复杂，先不介绍。
##### 角色(Actor)
系统角色，可以是人或者其他系统，子系统。以一个小人图标表示。
##### 对象(Object)
对象位于时序图的顶部,以一个矩形表示。这里的对象可以是包、类的实例等。

##### 生命线(LifeLine)
时序图中每个对象和底部中心都有一条垂直的虚线，这就是对象的生命线(对象的时间线)。以一条垂直的虚线表。

##### 控制焦点(Activation)
控制焦点代表时序图中在对象时间线上某段时期执行的操作。以一个很窄的矩形表示。

##### 消息(Message)
表现代表对象之间发送的信息。消息分为三种类型。
**同步消息(Synchronous Message)**
消息的发送者把控制传递给消息的接收者，然后停止活动，等待消息的接收者放弃或者返回控制。用来表示同步的意义。以一条实线+实心箭头表示。
 **异步消息(Asynchronous Message**)
消息发送者通过消息把信号传递给消息的接收者，然后继续自己的活动，不等待接受者返回消息或者控制。异步消息的接收者和发送者是并发工作的。以一条实线+大于号表示。
 **返回消息(Return Message)**
返回消息表示从过程调用返回。以小于号+虚线表示。

##### 自关联消息

表示方法的自身调用或者一个对象内的一个方法调用另外一个方法。以一个半闭合的长方形+下方实心剪头表示。

#### Obstacle Perception的时序图

![PlantUML diagram](http://www.plantuml.com/plantuml/png/PP5DJuH038Rl_HNDtatC7ZnmCCi7TqFqREhGh9Db1cm7DVvwTu1rG2uayBnzchwSD924qNMW5-i74dAe_36oDvoBz5_FxzPSFAlYSHMHVlIjwSMpuF5-1HnzErQbCwlzONn8B7cVI88rTY0VyAfXwQnd03AX5tnH5e1X5JdqRnh8T8m3Y-4XsDuVa1IRRUmVpRUqvS1nmSIABGj2vcBzUMBbbWdvCx-o1kleprrjCwPtRY4rGBm0xnwnoN0gDnvcFoamDW1D97csEWKJpo6FkCvmLCRPccWDCouod8z9g81YDnW_pRjk_IaAwzW5Q7arkLZB36yr82Pp2UNUN2xcDB4Nbr4yvQeCfn8nbPZy5LHB8uacMuhTPCsckdsmNh_aiUyu5Nz9XknRE5qVe-j-0G00)

**语法：**

```
@startuml
skinparam sequenceArrowThickness 2
skinparam roundcorner 20
skinparam maxmessagesize 60
skinparam sequenceParticipant underline

actor User

participant "obstacle perception" as R
participant "perception" as A
participant "detector" as B
participant "tracking" as C
participant "range_estimation" as D
participant "common" as E

User -> R: start
activate R

R -> A: perception 
activate A
A -> E: use common
activate E

A -> B: detect
activate B
B -> E: use common
B --> A: obstacles result
deactivate B

A -> C: tracking  
activate C
C -> E: use common
C --> A: tracking result
deactivate C

A --> D: range estimation
activate D
D -> A: estimation result
deactivate D
A --> R: perception result
deactivate E
deactivate A

R --> User: end
@enduml
```

### 活动图(Activity Diagram)描述

活动图可以用来描述算法或者关键函数的整个流程。活动图中包含选择、循环、分组等逻辑。以下我以detect为例，介绍一下如何描述一个关键函数的流程。

#### 代码

detect可以分为两个过程：

1. 从Feature-Map中解析出BBoxList。
2. 对BBoxList做处理，进行非极大值抑制，同时把它填入ObstacleList结构体中。

我对代码简单进行了一下重构，通过GetBBoxListFromFeature和GetObstacleListFromBBoxList这两个函数把这两块内容分离出来。

```
ObstacleList BaseDetector::Detect(const std::vector<float *> &result_blobs) {
  int mask_offset = 0;
  BBoxList bbox_list;
  for (unsigned int t = 0; t < result_blobs.size(); ++t) {
    GetBBoxListFromFeature(result_blobs[t], params_, t, mask_offset, &bbox_list);
    mask_offset +=  params_.num_anchors;
  }
  ObstacleList obstacle_list = GetObstacleListFromBBoxList(params_, &bbox_list);
  return obstacle_list;
}
```

#### 例子

活动图以实心圆为起点，活动用圆角框表示，六边形及反向箭头表示循环，以同心圆为结束。

![PlantUML diagram](http://www.plantuml.com/plantuml/png/FSt12i8m30RWUvwYHptu0Yles65U11zXj5l0Ocf6aZ8VttLitiAVd-zlrO9OoGJR0PUh4zH2DaJYg1u4PmpMtD6wZh-FfDOBvtxDYg276CRt4cHgqh_hbbSYTAVCWZlcAdOxGsMUSLqQ2G_gO7tTvlqvq9QeyGmjVgQIwGS0)

```
@startuml
start
-> result_blobs;
repeat:GetBBoxListFromFeature;
backward:is;
repeat while (more result blob?)
:GetObstacleListFromBBoxList;
-> obstacle_list;
stop
@enduml
```

## 附录

### 函数设计建议

我在写代码的时候时常会犯一些错误，如

1. 函数圈复杂度过高。
2. 函数名与函数的实际功能不一致。
3. 函数没有很好得被抽象（一个函数完成很多项任务）。
4. 一个函数运行经常会出现一些surprise。
5. 不知道函数用了哪些参数、改了哪些参数等

太困扰了，有什么办法呢？一个月前写的函数，我现在完全看不懂；程序总是模型其妙得崩溃；函数的扩展性差，总是要写一些重复性的代码。

#### 不知道函数用了哪些参数、改了哪些参数

以之间我重构的一个函数举例，最开始我是这么实现的：

```c++
void GetObstacleListFromBBoxList(ObstacleList obstacle_list, BBoxList bbox_list);
ObstacleList obstacle_list;
BBoxList bbox_list;
GetObstacleListFromBBoxlist(obstacle_list, bbox_list);
```

过了两个月，我已经不知道这段代码中，**使用的哪些参数**、**改了哪些数据**、**函数的输入输出**也不知道了。那有什么办法让我不要忘记这些吗？有！稍稍改动一下。

这个函数中我使用的参数是params_（DetectorParams） \  obstacle_list \ bbox_list，那把所有用到的参数体现在函数的参数列表中，**就不会忘记自己使用了哪些参数了**。

```
void GetObstacleListFromBBoxList(DetectorParams params，ObstacleList obstacle_list, BBoxList bbox_list);
GetObstacleListFromBBoxList(params_，obstacle_list, bbox_list);
```

我将不被修改的参数设置为const，把会被修改的参数设置为指针，那就不会忘记自己**修改了哪些参数**。

```
void GetObstacleListFromBBoxList(const DetectorParams &params，ObstacleList * obstacle_list, const BBoxList &bbox_list);
GetObstacleListFromBBoxList(params_，&obstacle_list, bbox_list)；
```

我还是不明确那个函数的输出会是什么？ 我可以把输出值通过函数值返回。

```
ObstacleList GetObstacleListFromBBoxList(const DetectorParams &params， const BBoxList &bbox_list);
ObstacleList obstacle_list = GetObstacleListFromBBoxList(params_， bbox_list);
```

现在这个函数就太清晰了，一眼就能知道这个函数干了什么事。

#### 函数没有很好得被抽象 && 函数名与函数的实际功能不一致 && 出现一些surprise

以为这个函数很完美了吗？不！之前讲到GetObstacleListFromBBoxList函数会对BBoxList做NMS，那么这个函数就一下子犯了三个错误：

1. 函数没有很好被抽象，好的抽象应该是一个函数只完成一个任务。
2. 函数完成的功能明显比函数名表达的多。
3. 我只是想从BBoxList中解析出ObstacleList，结果NMS也顺带做了，这真是一个不大不小的surprise。

```c++
ObstacleList GetObstacleListFromBBoxList(const DetectorParams &params， const BBoxList &bbox_list);
ObstacleList obstacle_list = GetObstacleListFromBBoxList(params_， bbox_list);
```

该怎么办呢？

s把函数分离出来就行了。ApplyNms不应该在GetObstacleListFromBBoxList中调用，而是应该在detect函数中调用。那么detect函数应该变为：

```c++
ObstacleList BaseDetector::Detect(const std::vector<float *> &result_blobs) {
  int mask_offset = 0;
  BBoxList bbox_list;
  for (unsigned int t = 0; t < result_blobs.size(); ++t) {
    GetBBoxListFromFeature(result_blobs[t], params_, t, mask_offset, &bbox_list);
    mask_offset +=  params_.num_anchors;
  }
  BBoxList bbox_list = ApplyNms(&bbox_list);
  ObstacleList obstacle_list = GetObstacleListFromBBoxList(params_, &bbox_list);
  return obstacle_list;
}
```



#### 函数圈复杂度过高

写代码的时候时常会遇到一些逻辑判断，而且有时候逻辑还复杂，可能会涉及三四层的逻辑判断。这时候我往往会看不懂两星期之前写的代码了，更不用说要做什么单元测试了。

前几天我在一个cutin的评测脚本，其中有一段代码大致如下：

```python
def get_cutin_flag(ov, label, pred):
    global COUNT, RANGE_CUTIN_STA_DICT
    COUNT += 1
    if ov < MINOVERLAP: 
        return False
    idx = str(int(pred[-1] / 10))
    if int(pred[-1] / 10) < 8:
        if(label[2] == pred[2]):
            if(label[2] == 2):
                return True
        	else:
            	return False    
        else:
            return False
          
    else:
        return False
```

过了两个星期，我开始不认识这段代码了，看了很久才看懂了这里的逻辑。有没有办法让它看起来清晰一些呢？一般两层包含条件语句会比较好理解，包含的条件语句越多就越复杂, 所以这里可以把一些条件判断抽象成一个函数，如：

```python
def is_match_cutin(label, pred):
    if(label[2] == pred[2]):
        if(label[2] == 2):
            return True
        else:
            return False    
    else:
        return False

def get_cutin_flag(ov, label, pred):
    global COUNT, RANGE_CUTIN_STA_DICT
    COUNT += 1
    if ov < MINOVERLAP: 
        return False
    idx = str(int(pred[-1] / 10))
    if int(pred[-1] / 10) < 8:
        is_match_cutin(label, pred)
    else:
        return False
```


