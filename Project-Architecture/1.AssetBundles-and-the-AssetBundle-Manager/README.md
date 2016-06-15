# AssetBundles and the AssetBundle Manager

## Introduction
AssetBundles允许从本地或者远程按需流化和加载资源。使用AssetBundles，资源可以在远端存储，当需要时被访问，从而提高啦项目的灵活性和减少了初始包的体积。

本课将会介绍AssetBundle和讨论1)如何使用，2)AssetBundle工作流步骤，3)如何把一个资源赋给一个AssetBundle，4)如何使用AssetBundle变体，5)如何构建和测试AssetBundle和变体。使用AssetBundle Manager 来简化创建，测试和部署ab。最后一部分将会使用简单的例子和脚本介绍几个加载和使用ab 和 ab变体的例子。

## Sample Project
在开始本教程前，最好先下载AssetBundle样例工程。

## What is an Assetbundle
AssetBundle是在编辑时编辑器中创建，在随后运行时中使用的文件。AssetBundle 可以包含诸如：模型，材质，纹理和场景等资源。但是AssetBundle不能包含脚本。

具体说来AssetBundle是在一个压缩文件中资源（或者场景）的集合，使得应用程序可以单独下载这些资源。AssetBundle可以在Unity中按需加载。允许流式和异步加载模型，纹理，音效片段，或者整个场景。AssetBundle可以"预先cache"在本地存放用来加快游戏第一次运行时的加载速度。然而ab最主要的目的是按需从远程流式内容，然后在必要的时候在应用中加载。ab可以包含任何Unity可以识别的资源类型，包括自定义的二进制类型。唯一的例外是脚本资源不支持。

ab有很多使用场景。新的资源可以被动态的加载和卸载。后期发布的DLC可以通过AB来轻松实现。通过资源在首次安装以后加载，应用的台面空间（体积）将会减少。不同平台和设备的资源可以在没有冗余资源的情况下加载各自的资源，或者不同分辨率的资源。"本地化" 将变得简单，可以通过根据不同的用户位置，语言和偏好设置来下载和安装资源。应用可以在不需要重新提交审核的情况下修复bug，修改，更新新内容。

资源的组织细节将严重依赖于各个项目，但是有些基本的信条需要理解：
* ab 都是整体下载和缓存
* ab 并不需要整体加载
* 在ab中的资源可能依赖其它资源
* 在ab中的资源可能和其它资源共享依赖
* 每个ab都有一定的技术开支，ab文件大小以及管理这些文件
* ab 应该为每个平台单独构建

ab 是整体下载的。如果ab中包含并不是立即需要的资源，即使这些资源并不需要加载进场景，他们也会占用下载带宽和存储空间。

ab 的内容并不需要整体加载，一旦ab已经下载好了，资源可以分开载入。

资源可能依赖其它资源。比如一个模型就可能有好多依赖。游戏中最终的模型并不只是mesh数据，它是一个包含所有组件和组件依赖的游戏对象。

这个坦克模型在模型的Mesh Renderer组件中依赖于一个材质资源，材质资源依赖于纹理的资源。而事实是，这个坦依赖了三个材质，而不仅仅一个。

资源之间可以共享依赖。比如，两个不同的模型可以共享同一个材质，这个材质依赖某个纹理。

两个岩柱有不同模型，但是共享同样的材质

每个ab都有一些技术开销。ab是包裹了资源的文件。这一层包裹增大了ab的体积。即使不是显著增大，但是也是不少。ab 也需要一些组织，创建，上传和维护的管理。更多的ab必然增加技术和管理上的开销。

当组织ab的时候，是选择比较多的小ab文件(额外的开销会比较小)，还是比较少的大ab，里面包含了不必要的冗余数据这两个是需要平衡的。这种平衡非常依赖于实际项目需求。

ab的内容根据每个目标平台针对倒入设置和当前目标平台有优化和编译。所以，每个不同的目标平台应该有单独的ab。

## Manifests and Dependency Managerment
关于依赖和依赖管理，有很多重要的点需要理解

资源依赖绝不会丢失。当依赖的资源没有被赋给一个ab的时候，依赖的资源将会自动添加到被选的资源中。这是非常方便，能够有效的防止依赖资源丢失。可是这可能会导致资源的重复。比如，两个岩柱共享同一个材质，如果两个岩柱在不同的ab中，然后材质并没有显式的赋给一个ab，那么这个材质将会被加到两个ab中。值得注意的是这时候，两个重复的资源存储在他们各自的ab中，然后ab之间的依赖就断了。每个模型资源将会依赖自己的材质资源，将不会再有共享材质资源的优势。解决这个问题，材质需要被显式的赋值给一个ab或者和其它资源共享一个ab。这样的话，岩柱的ab将会依赖于岩石材质的ab。

依赖和项目中其它ab信息保存在一个叫做Manifest的文件中。这个清单文件非常像项目ab的目录。当ab被构建好，Unity会产生大了数据。这些细节数据被存放在这个清单文件中。每个目标平台有一个清单文件。这个清单文件列出了所有当前构建平台的ab，以及所有它们的依赖。通过这个清单，查询ab和他们的依赖变得可能。

TODO
有一个非常特殊的设置叫做ab变体。ab变体设计出来为了支持需要对同一个对象有不同资源选择的情况。尤其适合在资源需要根据不同的分辨率，语言，位置信息做更改的情况。

# Working with AssetBundles and the AssetBundle Manager

## Introduction
使用ab其中一个费劲的地方就是构建和测试ab。经常在开发的时候，资源本身会有非常频繁的改动。正常情况下这就要求经常的构建ab，然后上传到服务器，然后通过网络连接在工程中测试ab。

本章节关注通过使用AssetBundleManger来使用ab。相比直接使用基础底层的API，AssetBundle Manager提供了比较高层次的API来改进工作流。

## Working with AssetBudnles
在编辑器中更实用ab可以大致分为以下步骤：1）编辑器中组织和设置ab;2)构建ab；3）上传ab到存储器中；4）运行时下载ab；5）从ab中加载对象。

值得注意的是，为了作为默认的设置立即加载某些ab可以一开始就在本地存储。这对于哪些不能连接到远程的存储的应用来说是一种保护。比如，当应用不能获取可下载的内容的时候，应用可以加载默认的语言和位置信息。

同样需要注意的是，一个保护平台信息的ab。ab的内容根据平台针对导入设置和目标平台构建和优化，所以，应该为每个平台构建ab。

在下面这个简单的场景中，一种比较合理的组织ab方式将是：一个基础的场景，保护地面，沙丘，岩柱，树木和仙人掌。这个场景可以包含依赖材质，因为它足够简单而且不会随着分辨率或者设备改变。坦克模型在他自己的ab中，这样就可以改变这个资源。两个额外的ab来完善坦克这个游戏对象，一个是依赖的材质，还有一个是依赖的纹理。这样就可以最小代价的更改纹理和材质。这个组织方式也允许在ab按需加载的时候改变版本或者变体。

在编辑器中组织和设置ab时，资源本身需要被赋值给ab。当预览一个资源的时候，ab 名称和变体名称可以在监视窗口下部看到。预览窗口必须要打开。

把一个资源赋给ab，使用下拉菜单。选择create new 或者 选择一个已经存在的。变体相关的在接下来讲述。

如果要创建一个新ab，选择new然后文本框就会自动激活用来输入一个新ab。

从一个ab中删掉一个资源，选择None，然后资源就会被反赋值。

想要从列表中删掉一个ab名字，所有赋给这个ab的资源必须先移除ab 名字，然后使用 Remove Unused Name。

资源将会被打包进选择的ab。ab名字必须严格的小写，如果使用了大写字母，Unity将会自动转化为小写。

## Using Assetbundle Variants
TODO

## Using the AssetBundle Manager
Unity 可以使用底层的API来直接操作ab。这个教程并不讲述底层的API，想要获得更多底层ab API 信息，请查阅底下的链接。

构建，测试，管理ab，本教程将会集中精力在AssetBundle Manager和他的高级API上。

AssetBundle Manager是一个可下载的包，可以在目前的Unity工程中安装，它提供了高级API和改进的ab工作流管理。AssetBundle Manager可以在这里下载。可以简单的添加AssetBundle Manager文件到工程Assets目录下来使用AssetBundle Manager。

在开发过程中构建和测试ab是非常痛苦的。资源经常会有改动。如果使用底层的ab API，测试需要重新构建ab 然后上传到远程，通过网络链接下载测试ab。AssetBundle帮助管理构建和测试ab的核心步骤。这个核心功能通过Simulation Mode，本地ab server和菜单选项 Build AssetBundles来无缝使用本地ab server。

把AssetBundle Manager添加到工程后会在Asset菜单中多出一个“AssetBundles”的选项。

选中AssetBundles菜单选项将会出现一个子菜单。

Simulation Mode，但打开的时候，允许编辑器模拟ab，而不用实际构建他们。通过选择Simulation Mode打开Simulation 模式。一个勾将会显示Simulation模式打开。再次选择菜单选项将会关闭Simulation 模式，勾也会被移除。

当Simulation模式打开的时候，编辑器查看哪些资源被赋值给ab，然后直接使用这些资源，就像他们在ab中狗一样。这些ab并不需要被构建。这样子就可以在编辑器中就像ab已经构建并且在远程一样开发。

当Simulation 模式打开的时候，带来的非常大的好处就是资源可以修改，操作，引入，删除只要他们位于某个ab中，在工程中开发夜并不需要停下来构建和部署ab。在Simulation Mode打开的时候测试是非常快的。

需要注意的是，ab变体并不能在simulation 模式下工作。如果想要测试ab变体，ab还是需要被构建和部署，ab变体可以通过Loacal Asset Server工作。

ABM同样可以打开Local Asset Server 从编辑器或者本地测试－包括手机。当Loacal Asset Server打开，ab必须构建好然后放在项目根目录中命名为"AssetBundles"的目录中（目录层级和Assets一样）

到ab 在本地后，通过少量代码就能访问本地资源服务器。请阅读AssetBundle Sample工程，教程接下来也会涉及。

可以简单的通过选择 Assets/AssetBundles/Build AssetBundles 来构建和保存ab到 AssetBundle目录。当“Build AssetBuilds”被选中时，Unity将会构建所有ab，针对当前的目标平台编译优化他们，最后在项目根目录的AssetBundles目录保存他们以及一个主要的清单文件。如果不存在 AssetBundles 目录，Unity将会创建一个，在AssetBundls目录里面，按照平台组织ab。

当AssetBundles构建以及部署好以后或者打开本地ab服务器，这些ab就可以在运行时被下载然后合并。

## Using AssetBundles in Practice
教程将会讲述如何在实践中使用 AssetBundle Manager。 AssetBundle Manager将会解决ab的加载和它们依赖资源的加载。如果是使用AssetBundle Manager来从ab中加载资源，就必须使用AssetBundle Manager 提供的API。

AssetBundle Manager提供的API包括：
* Initialize() 初始化 ab manifest 对象
* LoadAssetAsync() 从给定的ab中加载给定的资源，并且处理好依赖
* LoadLevelAsync() 从给定的ab中加载给定的场景，并且处理好依赖
* LoadDependecies() 加载给定ab的依赖ab
* BaseDownloadingURL 当需要自动下载依赖的时候，设置一个基础的下载url
* SimulateAssetBundleInEditor 设置Simulation 模式
* Variants 设置变体
* RemapVariantName() 根据当前变体，处理好ab

在AssetBundle Manager工程中 AssetBundle Sample 目录下有一些样例文件。 在AssetBundleSample/Scenes目录下有三个基础的样例场景和一个稍微高级一点的样例场景。

* AssetLoader 演示了如何从ab中加载一个普通资源
* SceneLoader 演示了如何从ab中加载一个场景
* VariantLoader 演示了如何加载一个ab变体
* LoadTanks 稍微高级一点，演示了稍微复杂的情况，在同一个场景中加载场景，资源以及变体

每个场景都单独有一个脚本：LoadAssets.cs, LoadScenes.cs, LoadVariants.cs 和 LoadTanks.cs

需要再次重申AssetBundle Manager 提供的工作流

有三种情况，成功的测试完成ab

第一种情况，不使用AssetBundle Manager，将需要构建和部署，测试一系列完成的系统。第二种情况，工程中每个资源的变更，新的ab就需要构建和部署。

使用AssetBundle Manager，有两个改进之处。他们就是本地ab服务器和Simulation 模式。

在Simulation 模式中，AssetBundle Manager 模拟 已经构建的ab。这是最快的工作流。只需要打开 Simulation 模式然后就可以测试工程了。没有ab将会被构建。需要注意的是，ab变体不能在Simulation 模式中工作。同样需要注意的是，资源可以在工程中被操作，结果还是可以在场景中体现的，如果已经部署上去了，那是不可能的事情。

本地ab服务器提供了更加精确的ab部署，但是这需要构建ab，然后把他们保存在项目默认的目录中。当本地ab服务开启的时候，构建好的ab可以在编辑器中通过本地网络获取。注意这是测试ab变体唯一的方法。

任意一个样例场景，AssetBundle Manager 必须跑在其中一个模式上。跑AssetBundle 变体场景，ab必须构建好然后本地ab服务器必须打开。

## Example 1: Loading Assets
* 使用菜单选项打开Simulation 模式
* 打开 AssetLoader 场景
* 注意场景是空的，只有 Main Camera, Directional Light 和 Loader 游戏对象。
* 点击 Play 
* 可以看到cube从ab中加载到场景

这个场景被LoadAssets 脚本驱动

打开AssetBundleSample/Scripts/LoadAssets.cs

有两个public变量， public string assetBundleName 和 public string assetName;

* public string assetBundleName 需要加载的ab的名字
* public string assetName 需要从ab中加载的资源的名字。
 
脚本由一个Start函数和两个由Start调用的协程组成。在 Initialize()， DontDestroyOnLoad() 被调用，AssetBundle的路径被设置以及AssetBundle 清单初始化。在 InstantiateGameObjectAsync() 中资源和ab名称被AssetBundleManager.LoadAssetAsync() 请求如果这个资源请求不为空，那么它将被构造出来。

需要被重视的是，查看工程中的MyCube 资源可以看到，MyCube是依赖于MyMaterial，MyMaterial依赖于UnityLogo纹理，虽然只有MyCube被请求，但是所有的依赖资源也会被正确的加载。

需要注意的是AssetBundle的路径是如何被指定的。当运行在编辑器或者Development build 版本的时候AssetBundle的位置将会赋值给本地ab服务器。当simulation 模式打开的时候，ab将会被模拟这个设置将不会使用。

需要理解一下 DontDestroyOnLoad() ，即使在这个场景中它非常简单，它的出现代表了把脚本作为ab加载器，而不会随着场景的切换而变化。

## Example 2: Loading Scenes
* 确保Simulation 模式打开
* 打开 SceneLoader 场景
* 注意场景是空的，只有 Main Camera, Directional Light 和 Loader 游戏对象。
* 点击 Play 
* 可以看到cube和一个平面加载进来。

这个场景被LoadScene.cs 脚本驱动

有两个public变量： public string sceneAssetBundles和public string sceneName

* sceneAssetBundles 需要加载的ab名字
* sceneName 需要从ab加载的场景名字

这个脚本包含一个Start函数以及两个由Start调用的协程。在Initialize函数中DontDestroyOnLoad被调用，ab的路径和ab 清单被初始化。在InitializeLevelAsync 中场景名字和是否添加两个参数被用来请求一个场景。如果场景请求是null，那么AssetBundle Manage将会在控制台答应错误以及结束协程。

需要注意的是，查看工程中的MyCube可以看到，Cube依赖于MyMaterial材质，材质依赖于UnityLogo纹理。只有TestScene被请求，Cube包含在TestScene里面，所有依赖资源都被AssetBundle Manager正确的载入。

需要注意的是AssetBundle的路径是如何被指定的。当运行在编辑器或者Development build 版本的时候AssetBundle的位置将会赋值给本地ab服务器。当simulation 模式打开的时候，ab将会被模拟这个设置将不会使用。

需要理解一下 DontDestroyOnLoad() ，即使在这个场景中它非常简单，它的出现代表了把脚本作为ab加载器，而不会随着场景的切换而变化。

## Example 4: Tanks Example
这个更复杂的例子将会把教程中所有的都涉及，包括从ab中加载场景和为不同的分辨率,内容，位置加载ab变体。
* 确保Simulaton 模式是关闭的
* 确保本地ab服务器是打开的
* 打开TanksLoader场景
* 注意这个场景只包含一个Loader游戏对象
* 点击 play
* 选择分辨率，皮肤，语言
* 加载的资源根据这里的选择
* 注意如果没有显式的选择，AssetBundle Manager将会自动选择然后在控制台答应出警告

这个场景由LoadTanks.cs 驱动。

在编辑器中打开LoadTank.cs

这个脚本非常类似LoadScenes.cs 和 LoadAsset.cs。这个脚本用了依赖变体的加载场景和加载额外依赖变体的游戏对象。还有一些用来创建UI的代码。
* public string sceneAssetBundle 承载场景的ab名字
* public string sceneName 从ab中加载的场景名称
* public string textAssetBundle 承载文本资源的ab名字
* public string textAssetName 从ab中加载的文本资源名称
* private string[] activeVariants 传给AssetBundleManager的变体
* private bool bundlesLoaded 标记当资源加载的时候隐藏UI
* private bool sd, hd, normal, desert,english, danish 用来设置变体
* private string tankAlbedoStyle, tankAlbedoResolution, language, 用来设置变体

这个脚本包含一个BeginExample 和三个由它调用的协程。BeginExample() 函数在按下 OnGUI() 函数中的 Load Scene 按钮后调用。在Initialize函数DontDestroyOnLoad() 函数被调用，ab的路径以及ab的manifest被初始化。在BeginExample函数中，Initialize() 和 InitializeLevelAsync() 之间设置了变体。变体变量的值是在OnGui函数中根据用户输入确定的。在InitializeLevelAsync() 函数中 场景名称和是否添加用来使用AssetBundleManager.LoadLevelAysync请求。如果场景请求是空的，那么AssetBundle Mananger 将会在控制台打印出error然后协程停止。在InstantiateGameObjectAsync() 函数中资源和ab名字作为请求，如果资源清酒不为空，对象就加载出来。如果ab不能被加载或者资源不能请求，控制台将会打印出错误。

需要注意的是这些资源，ab和ab变体是如何被访问和加载到场景中的，以及如何在运行时设置它们的值。
