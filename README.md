# layui-tableTreeDj

#### 介绍
使表格增加了树形结构展示的能力.完全依赖于layui的表格.
正因为如此,您可以像使用表格组件一样使用该组件.layui的表格功能全都有.全都有.全都有.
注意: 本组件对数据有要求.要求父节点的ID小于子节点.否则会显示异常.

### 重要：虽然官方的table支持分页，但是使用本扩展将其扩展为支持树的table时，强烈不建议使用分页。毕竟整个树分页，不好界定分页条件，会导致数据隔断，从而出现分页后，原本应为第二级的，因为找不到父级二成为了第一级的情况发生。

#### 组件引入方法请阅读 官方文档
https://www.layui.com/doc/base/modules.html#extend

#### 方法
* render(): 表格渲染.一般第一次显示调用.或者其他操作比如删除/添加等操作后也可以调用.第二次调用可以不传参数.如果传递参数会将上次参数覆盖.
* reload(): 表格重载,内部调用了table.reload().一般用于搜索后显示数据.提交where条件给后端.
* getTable(): 由于该组件内部使用了layui.table.如果想更细粒度的操作table.可以使用此方法获取table对象
* 其他方法: 请阅读源码,只要方法名不以下划线开头都可以使用.如果需要的话.


#### 参数
* keyId: 数据ID.一般对应数据库的主键.默认: 'id'
* keyPid: 数据父ID,与父级数据的ID相等.此参数与ID确定上下级关系.默认: 'pid'
* title: 泛指数据标题,来自列(cols)的field属性.表明此字段在被点击时候会展开/折叠下级. 默认: 'name'
* indent: 缩进字符.在此设置的字符会添加到title列前面.次数为与层级的乘积.默认: ' &nbsp; &nbsp;'
* icon.open: 标题前面的小图标.在展开时候显示.是css的class属性字符串.可以包含多个类,用空格隔开.默认: 'layui-icon layui-icon-triangle-d'
* icon.close: 标题前面的小图标.同open.在折叠时候显示.默认: 'layui-icon layui-icon-triangle-r'
* showCache: 这里就要好好说说了.数据展开折叠缓存.这个配置会影响组件在渲染时候的行为.默认为false.
    - 如果传false表示不使用缓存.此时渲染完成的状态为全部折叠; 
    - 如果传true.会把操作过程中的展开折叠状态记录到 localStorage 中.key为 unfoldStatus; 
    - 可以传一个字符串.这时候与传true类似,区别是 localStorage 的 key 为传入的字符串.建议传字符串.可以有效避免多个页面之间的冲突.
* sort: pid排序方式,可选值为 asc / desc, 默认 asc.必须小写.会影响所有层级.
* layui的table的参数 initSort: {field: 'sort', type: 'desc'}，为第二排序功能，影响同级别的对象排序
以上参数都可以不传(在与默认值完全一致的情况下).

#### 特别说明
很遗憾.为了实现点击折叠展开功能.我给表格的某个字段绑定了点击事件.但是除此之外我不知道该怎么做才能做到不污染原始表格.可能会给您带来困扰.建议将 title 字段设置为用户平时不点击的字段以期将影响降低到最小

---

被占用名称的列即上述参数的 title 列，若设置为name，在layui的table的tool事件中，可通过 `data[__name__]` 的形式获取原始数据。这在编辑时比较有用。

---

在layui的table的tool事件中，获取的data可以通过 `__hasChild__` 的形式获取该数据是否还有子级，true-有子级，false-没有子级


#### 代码示例
```javascript

layui.use(['tableTreeDj'], function() {
        const tableTree = layui.tableTreeDj;
        const $ = layui.$;

        // 与 layui.table 的参数完全一致,内部本来就是把这些参数传递给table模块的
        const objTable = {
            elem: '#test'
            // ,url: "./getData" //url与data二选一，优先url
          , data:[
            {"id": "b26a7bed26645039864a58fc9530de48", "pid": "0", "name": "顶层1",open:true},//参数open=true表示此节点默认展开
            {"id": "c32a7bed26645039864c32fc9530de32", "pid": "0", "name": "顶层2"},
            {"id": "5a812646ecf25d6c9f1951320f5f15df", "pid": "b26a7bed26645039864a58fc9530de48", "name": "第一层aaa",sort:5},
            {"id": "9fd3dcc30ec95f059d866e66f30de1d4", "pid": "b26a7bed26645039864a58fc9530de48", "name": "第一层000",sort:20},
            {"id": "9acb151537f6555d918f8059c196198c", "pid": "b26a7bed26645039864a58fc9530de48", "name": "第一层bbb",sort:3},
            {"id": "a913db2f99775b1098ae5621b699c091", "pid": "5a812646ecf25d6c9f1951320f5f15df", "name": "aaa111",sort:4},
            {"id": "1b2945c5474756bda51294bd59c28ad0", "pid": "a913db2f99775b1098ae5621b699c091", "name": "a10",sort:10}
          ]
            ,cols: [[
              {field: 'name', title: '名称'}
              , {field: 'sort', title: 'SORT'}
              , {field: 'id', title: 'ID'}
              , {field: 'pid', title: '上级ID'}
            ]]
            ,id:'list'
            , initSort: {field: 'sort', type: 'desc'}//设置同级别对象按sort值排序，优先级高于下面的config.sort。上面数据中，第一层aaa、第一层000、第一层bbb 将按照 第一层000、第一层aaa、第一层bbb 的顺序展示
        }

        // 本组件用到的参数, 组件内部有默认值,与此一致,因此您可以只声明不一致的配置项
        const config = {
            keyId: "id" // 当前ID
            , sort: 'desc'//pid排序方式。上面数据中，顶层1、顶层2 将按照 顶层2、顶层1 的顺序展示
            , keyPid: "pid" // 上级ID
            , title: "name" // 标题名称字段,此字段td用于绑定单击折叠展开功能
            , indent: ' &nbsp; &nbsp;' // 子级td的缩进.可以是其他字符
            // 图标
            , icon: {
                open: 'layui-icon layui-icon-triangle-d', // 展开时候图标
                close: 'layui-icon layui-icon-triangle-r', // 折叠时候图标
            }
            , showCache: true //是否开启展开折叠缓存,默认不开启.
        };
        // 渲染 
        tableTree.render(objTable, config);

        // 其他一系列操作后.重新渲染表格,此时可以不传递参数.内部记录了上次传入的参数.
        tableTree.render();
        
        // 点击搜索按钮后重载数据.此时可以传入where条件.obj参数与官方表格一致.
        obj = {where:{id: 1}};
        tableTree.reload(obj);


    });
```
