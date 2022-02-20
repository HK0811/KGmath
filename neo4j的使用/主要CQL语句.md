1知识图谱生成大致流程：
    1.1整理好的excel表格数据转换成csv文件，保存在neo4j安装目录的import文件下
    1.2使用neo4j自带的load方法直接导入，分别生成知识点节点和关系节点
    1.3根据指定条件查询关系节点和知识点节点，生成关系，即得到知识图谱KG11(一年级上册知识图),KG12(一年级下册知识图)以及KG（整个小学数学的知识图）
    1.4再根据需求对图谱进行修改得到最终图
    
2.一些节点的命名规则
    2.1形如KG11，KG是知识图谱缩写，11分别指代年级和学期，11就是一年级上学期，21就是二年级上学期
    2.2KG是整个小学数学的知识图，KGlong是在KG的基础上仅保留最长路径而得到
    
3.主要的CQL语句

   3.1创建知识图谱
        //以创建一年级下册知识图为例，其他年级类似
        //先导入文件一年级下册投票结果
        load csv from"file:///1下投票结果.csv"as line
        create (:pr12{from:line[0],to:line[1],approval:line[2]})
        //生成一年级下册的知识图
        match (m:knowledgepoint),(n:knowledgepoint),(r:pr12)
        where m.name=r.from and n.name=r.to
        create (m)-[:KG12{approval:r.approval,blank:'-'}]->(n)
        return m
        
   3.2创建小学知识图谱
        load csv from"file:///小学投票结果.csv"as line
        create (:pr{from:line[0],to:line[1],approval:line[2]})
        match (m:knowledgepoint),(n:knowledgepoint),(r:pr)
        where m.name=r.from and n.name=r.to
        create (m)-[:KG{approval:r.approval,blank:'-'}]->(n)
        return m
        
4.KG到KGlong：仁义节点之间保留最长路径，删除短路径
    删除和保留的原则：
        （1）已经连通的节点之间必须保证依然连通
        （2）选择最长的连通路径进行保留
        （3）原则（1）的优先级高于原则2的优先级
        

