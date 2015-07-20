title: Hello World
---

Markdown 编辑器：mdcharm(http://www.mdcharm.com/)

Markdown 语法：http://wowubuntu.com/markdown/

表格语法：

```
|宏名                       |含义|
|---                        |:---|
|LW_OPTION_NOT_WAIT         |不等待立即退出|
|LW_OPTION_WAIT_INFINITE    |永远等待|
|LW_OPTION_WAIT_A_TICK      |等待一个时钟嘀嗒|
|LW_OPTION_WAIT_A_SECOND    |等待一秒|
```

表格效果：

|宏名                       |含义|
|---                        |:---|
|LW_OPTION_NOT_WAIT         |不等待立即退出|
|LW_OPTION_WAIT_INFINITE    |永远等待|
|LW_OPTION_WAIT_A_TICK      |等待一个时钟嘀嗒|
|LW_OPTION_WAIT_A_SECOND    |等待一秒|

plantuml 语法：http://plantuml.sourceforge.net/

PC 上使用 Eclipse 插件 plantuml - http://plantuml.sourceforge.net/updatesite/ 

plantuml 生成 UML 图的例子：

```
{% plantuml %}

@startuml
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
@enduml

{% endplantuml %}
```

{% plantuml %}

@startuml
Alice -> Bob: Authentication Request
Bob --> Alice: Authentication Response

Alice -> Bob: Another authentication Request
Alice <-- Bob: another authentication Response
@enduml

{% endplantuml %}

plantuml 使用 DOT 语法生成流程图的例子：

```
{% plantuml %}

@startuml
digraph G {
    subgraph cluster0 {
        node [style=filled,color=white];
        style=filled;
        color=lightgrey;
        a0 -> a1 -> a2 -> a3;
        label = "process #1";
    }
    subgraph cluster1 {
        node [style=filled];
        b0 -> b1 -> b2 -> b3;
        label = "process #2";
        color=blue
    }
    start -> a0;
    start -> b0;
    a1 -> b3;
    b2 -> a3;
    a3 -> a0;
    a3 -> end;
    b3 -> end;
    start [shape=Mdiamond];
    end [shape=Msquare];
}
@enduml

{% endplantuml %}
```

{% plantuml %}

@startuml
digraph G {
    subgraph cluster0 {
        node [style=filled,color=white];
        style=filled;
        color=lightgrey;
        a0 -> a1 -> a2 -> a3;
        label = "process #1";
    }
    subgraph cluster1 {
        node [style=filled];
        b0 -> b1 -> b2 -> b3;
        label = "process #2";
        color=blue
    }
    start -> a0;
    start -> b0;
    a1 -> b3;
    b2 -> a3;
    a3 -> a0;
    a3 -> end;
    b3 -> end;
    start [shape=Mdiamond];
    end [shape=Msquare];
}
@enduml

{% endplantuml %}

也可以使用 Graphviz http://www.graphviz.org/ ，而不使用 plantuml 插件，但只能用 DOT 语法。
