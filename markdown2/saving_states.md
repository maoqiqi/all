# Saving UI States

跨系统发起的活动或应用程序破坏及时保存和恢复活动的UI状态是用户体验的一个重要部分。在这些情况下，用户希望UI状态保持不变，但是系统会销毁活动和其中存储的任何状态。

要在用户期望和系统行为之间架起桥梁，可以使用ViewModel对象、onSaveInstanceState()方法和/或本地存储的组合来跨应用程序和活动实例转换持久化UI状态。
决定如何组合这些选项取决于UI数据的复杂性、应用程序的用例以及检索速度和内存使用情况。

无论采用哪种方法，都应该确保应用程序满足用户对UI状态的期望，并提供流畅、快速的UI(避免在将数据加载到UI时的延迟，尤其是在频繁发生配置更改(如旋转)之后)。
在大多数情况下，应该同时使用ViewModel和onSaveInstanceState()。


## 目录

* [用户期望和系统行为](#用户期望和系统行为)
  * [用户发起的UI状态终止](#用户发起的UI状态终止)
  * [系统启动的UI状态终止](#系统启动的UI状态终止)
* [用于保留UI状态的选项](#用于保留UI状态的选项)
* [使用ViewModel处理配置更改](#使用ViewModel处理配置更改)
* [使用onSaveInstanceState来处理系统启动的进程死亡](#使用onSaveInstanceState来处理系统启动的进程死亡)
* [使用本地持久性来处理复杂或大型数据的进程死亡](#使用本地持久性来处理复杂或大型数据的进程死亡)
* [管理UI状态](#管理UI状态)
* [恢复复杂状态](#恢复复杂状态)


## 用户期望和系统行为

根据用户所采取的操作，他们要么希望清除活动状态，要么希望保留状态。在某些情况下，系统会自动执行用户所期望的操作。在其他情况下，系统执行与用户期望相反的操作。

### 用户发起的UI状态终止

用户希望，当他们启动一个活动时，该活动的临时UI状态将保持不变，直到用户完全取消该活动。用户可以完全取消一个活动:

* 按下后退按钮
* 从概览（最近）屏幕上滑动活动
* 从活动中导航
* 从"设置"屏幕中删除该应用
* 完成某种"完成"活动(由activity .finish()支持)

在这些完全解雇案例中，用户的假设是他们已经永久地从活动中导航出去，如果他们重新打开该活动，他们希望该活动从一个干净的状态开始。
这些终止场景的底层系统行为符合用户的期望——活动实例将被销毁并从内存中删除，同时删除存储在其中的任何状态以及与活动关联的任何保存的实例状态记录。

这条关于完全删除的规则也有一些例外——例如，用户可能希望浏览器在使用后退按钮退出浏览器之前将他们带到他们正在查看的确切页面。

### 系统启动的UI状态终止

用户期望活动的UI状态在整个配置更改期间保持不变，例如旋转或切换到多窗口模式。但是，默认情况下，系统会在发生此类配置更改时销毁活动，从而消除存储在活动实例中的任何UI状态。
请注意，可以（但不建议）覆盖配置更改的默认行为。有关更多详细信息，请参阅[Handling the Configuration Change Yourself(自己处理配置更改)](runtime_changes.md#自行处理配置变更)。

如果用户暂时切换到其他应用程序然后稍后返回您的应用程序，则用户还希望您的活动的UI状态保持不变。
例如，用户在您的搜索活动中执行搜索，然后按下主页按钮或接听电话。当他们返回搜索活动时，他们希望找到搜索关键字并且结果仍然存在，就像之前一样。

在这种情况下，您的应用程序将放置在后台，系统会尽最大努力将您的应用程序进程保留在内存中。但是，当用户离开与其他应用程序交互时，系统可能会破坏应用程序进程。
在这种情况下，活动实例以及存储在其中的任何状态都将被销毁。当用户重新启动应用程序时，活动意外处于干净状态。
要了解有关进程死亡的更多信息，请参阅[Processes and Application Lifecycle(进程和应用程序生命周期)](process_lifecycle.md)。


## 用于保留UI状态的选项

当用户对UI状态的期望与默认系统行为不匹配时，必须保存并恢复用户的UI状态，以确保系统发起的破坏对用户是透明的。

保存UI状态的每个选项都在以下影响用户体验的维度上有所不同:

|选项|ViewModel|onSaveInstanceState|持久存储|
|:-------|:-------|:-------|:-------|
|存储位置|内存|序列化到磁盘|在磁盘或网络上
|幸存配置更改|Yes|Yes|Yes|
|幸存系统启动的过程死亡|No|Yes|Yes|
|用户完成活动终止/onFinish()|No|No|Yes|
|数据的局限性|复杂的对象很好，但是空间受到可用内存的限制|仅适用于基本类型和简单的小对象，如String|仅受磁盘空间或从网络资源检索的成本/时间的限制|
|读/写时间|快速(仅限内存访问)|慢(需要序列化/反序列化和磁盘访问)|慢(需要磁盘访问或网络事务)|


## 使用ViewModel处理配置更改

当用户积极使用应用程序时，ViewModel是存储和管理ui相关数据的理想工具。
它允许快速访问UI数据，并帮助您避免通过旋转、窗口大小调整和其他常见的配置更改从网络或磁盘重新获取数据。
要了解如何实现ViewModel，请参阅[ViewModel](view_model.md)指南。

ViewModel将数据保留在内存中，这意味着它比磁盘或网络中的数据更便宜。
ViewModel与活动（或其他一些生命周期所有者）相关联，它在配置更改期间保留在内存中，系统自动将ViewModel与配置更改产生的新活动实例相关联。

当用户退出活动或fragment或调用finish()时，系统会自动销毁ViewModel，这意味着状态将在用户期望的情况下被清除。

与保存的实例状态不同，ViewModel在系统启动的进程死亡期间被销毁。这就是为什么您应该结合使用ViewModel对象
和onSaveInstanceState()(或其他一些磁盘持久性)，将标识符保存在savedInstanceState中，以帮助视图模型在系统死后重新加载数据。


## 使用onSaveInstanceState来处理系统启动的进程死亡

如果系统销毁并稍后重新创建该控制器，onSaveInstanceState()回调函数存储了重新加载UI控制器状态(如活动或片段)所需的数据。

保存的实例状态包同时保存配置更改和进程终止，但是由于onSavedInstanceState()将数据序列化到磁盘，所以受到存储量和速度的限制。
如果被序列化的对象很复杂，序列化会占用大量内存。由于此过程在配置更改期间在主线程上发生，因此序列化可能会导致丢帧和视觉卡顿（如果耗时太长）。

不要使用store onSavedInstanceState（）来存储大量数据（如位图），也不要使用需要冗长序列化或反序列化的复杂数据结构。
相反，只存储基本类型和简单的小对象，如String。因此，使用onSaveInstanceState()存储少量必要的数据，
例如ID，以便在其他持久性机制失败时重新创建必要的数据，将UI恢复到以前的状态。大多数应用程序应该实现onSaveInstanceState（）来处理系统启动的进程死亡。

根据应用程序的用例，您可能根本不需要使用onSaveInstanceState()。例如，浏览器可能会将用户带回他们退出浏览器之前正在查看的准确页面。
如果您的活动以这种方式运行，您可以放弃使用onSaveInstanceState()，而是在本地保存所有内容。

在上述任一情况下，您仍然应该使用ViewModel，以避免在配置更改期间浪费从数据库重新加载数据的周期。

此外，当您从intent打开活动时，会在配置更改和系统恢复活动时将附加捆绑包传递给活动。如果在启动活动时将一个UI状态
数据（例如搜索查询）作为intent extra传递，则可以使用extras bundle而不是onSaveInstanceState（）包。

在需要保存的UI数据简单且轻量级的情况下，可以单独使用onSaveInstanceState()来保存状态数据。


## 使用本地持久性来处理复杂或大型数据的进程死亡

只要您的应用程序安装在用户的设备上(除非用户为您的应用程序清除了数据)，持久的本地存储(如数据库或共享首选项)就会一直存在。
虽然这种本地存储在系统启动的活动和应用程序进程终止后仍然存在，但是检索它的代价可能很高，因为它必须从本地存储读取到内存中。
通常，这种持久的本地存储可能已经是应用程序体系结构的一部分，用于存储在打开和关闭活动时不想丢失的所有数据。

ViewModel和保存的实例状态都不是长期存储解决方案，因此不能替代本地存储，比如数据库。相反，您应该使用这些机制来临时存储临时UI状态，并为其他应用程序数据使用持久存储。


## 管理UI状态

通过将工作划分到各种类型的持久性机制中，可以有效地保存和恢复UI状态。在大多数情况下，根据数据复杂性、访问速度和生命周期的权衡，每种机制都应该存储活动中使用的不同类型的数据:

* ViewModel：在内存中存储显示关联UI控制器所需的所有数据。
  * 示例：最近搜索的歌曲对象和最近的搜索查询。
* onSaveInstanceState():存储少量数据，以便在系统停止时轻松重新加载活动状态，然后重新创建UI控制器。
  与其在这里存储复杂对象，不如将复杂对象持久存储在本地存储中，并在onSaveInstanceState()中存储这些对象的惟一ID。
  示例:存储最近的搜索查询。
* 本地持久性:存储您不希望在打开和关闭活动时丢失的所有数据。
  * 示例:歌曲对象的集合，其中可能包括音频文件和元数据。

例如，考虑一个允许您搜索歌曲库的活动。以下是处理不同事件的方法:

当用户添加一首歌时，ViewModel立即委托本地存储该数据。如果这个新添加的歌曲应该显示在UI中，
那么还应该更新ViewModel对象中的数据，以反映添加的歌曲。记得从主线程中删除所有数据库插件。

当用户搜索一首歌时，从数据库中为UI控制器加载的任何复杂歌曲数据都应该立即存储在ViewModel对象中。您还应该将搜索查询本身保存在ViewModel对象中。


## 恢复复杂状态

当用户返回到该活动时，有两种可能的场景用于重新创建该活动:

* 该活动在系统停止后被重新创建。该活动将查询保存在onSaveInstanceState()包中，并应将查询传递给ViewModel。
  ViewModel看到它没有缓存搜索结果，并使用给定的搜索查询委托加载搜索结果。
* 该活动是在配置更改之后创建的。该活动将查询保存在onSaveInstanceState()包中，并且ViewModel已经缓存了搜索结果。
  您将查询从onSaveInstanceState()包传递到ViewModel，该视图模型确定它已经加载了必要的数据，并且不需要重新查询数据库。

> **注意**:当一个活动最初被创建时，onSaveInstanceState()包不包含任何数据，并且ViewModel对象是空的。
创建ViewModel对象时，传递一个空查询，该查询告诉ViewModel对象还没有要加载的数据。因此，活动以空状态启动。