---
title: 'PIT 编辑器编辑及协同架构说明'
date: 2019-12-02 21:15:00
cover: false
---
pit 项目使用 quill-delta 作为数据层存储文档内容数据，quill-delta 是一个基于 OT 算法的库，用 quill-delta 作为数据层，不仅能很好的保存文档数据，还可以方便的实现文档的协同编辑，即多个人同时编辑同一份文档（需要服务器支持）。

quill-delta 数据格式不仅能很好的描述完整的文档内容，还可以很方便的描述文档的修改过程，所以 pit 在进行架构设计的时候，并不仅仅考虑单机编辑的情况，同时还考虑到了协同编辑的情况，以方便以后在需要的时候实现协同编辑功能。

![](450824-20191202211131616-416895541.png)

上图即为 pit 编辑器单机使用时的系统架构。基本流程是用户通过各类 action 修改数据 model，比如用户要在某个位置插入一个字符“A”，就会直接通过 insert 接口在 model 中插入 “A” 这个字符，然后编辑器发现某个 model 更新后，会调用其 layout 方法对内容进行重新排版，然后用 render 方法把排版后的内容绘制到 view 上，让用户可以看到自己刚刚插入的内容。

同时 model 会用 quill-delta 的 diff 算法计算出文档内容修改前后的数据差异，并通过 delta 的方式来表示这种差异，即为 Delta Diff。

Delta Diff 代表了用户刚刚对文档进行的修改。比如原文档内容为

![](450824-20191202211202258-2017096908.png)

用户在“好”和“世”之间插入一个字符“A”，则产生的 Delta Diff 为

![](450824-20191202211210924-119546379.png)

即光标从 0 位置向后移动两个位置，然后插入字符“A”。

Delta Diff 代表了用户此次对文档内容的操作，通过 quill-delta 的 invert 接口即可产生一个代表相反操作的 Invert Delta Diff，如上面的 Delta Diff 在 invert 操作之后即可得到

![](450824-20191202211221339-1313893550.png)

即光标从 0 位置向后移动两个位置，然后删除一个字符。

将 Delta Diff 和 Invert Delta Diff 作为一组一起压入 History Stack 并设置一个指向当前操作的游标，即可实现文档编辑器上常见的 redo、undo 功能。

![](450824-20191202211259077-1141801955.png)

进行协同编辑的场景和单机编辑的场景基本类似，区别仅在于用户修改文档产生了 Delta Diff 之后，将这个 Delta Diff 通过服务器中转发给其他一起进行协同编辑的客户端（上图中的 User B），这个用户就可以通过收到的 Delta Diff 来完成和 User A 完全相同的对 Model 的操作，并渲染到 View 上，这样就基本实现了协同编辑。User B 的编辑器在收到 Delta Diff 后，先判断收到的 Delta Diff 会对文档中的那些段落产生影响，然后获取这些段落的 Delta 数据，并将 Delta Diff 合并到 Delta 中，这样就得到了代表这些段落新状态的 Delta，这时再通过 readFromDelta 方法即可由新的 Delta 产生新的段落 Model，然后通知编辑器重新排版并渲染到用户界面上。

具体实现可参考：[https://github.com/SilentTiger/pit/releases/tag/draft-03](https://github.com/SilentTiger/pit/releases/tag/draft-03)