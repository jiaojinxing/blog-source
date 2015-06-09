title: Hello World
---

Markdown 编辑器：https://stackedit.io/editor   https://www.zybuluo.com/mdeditor（在线） 

Markdown 语法：http://wowubuntu.com/markdown/（中文）

github page 博客不支持的 Markdown 编辑器语法有: 注脚、flow 流程图、seq 序列图

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
