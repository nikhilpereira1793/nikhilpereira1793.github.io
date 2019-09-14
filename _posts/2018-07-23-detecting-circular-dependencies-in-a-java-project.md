---
tags: [java, code maintenance]
---

As Java projects get bigger, with several developers working on them, it may lead to code having circular
dependencies. It is difficult to serialize objects with circular dependencies. Below is a method to detect circular dependencies in a Java project.

### Program Goal
- **Input** : Path to Java project source files
- **Output** : Folder containing image files of cyclic graphs for each class having circular dependencies

---

### Steps
  - Get a list of all the class names in a Java project
  - Using [Java Parser](https://github.com/javaparser/javaparser), create a Map of all the instance variables types belonging to a class. Filter the list of instance variable types against the list of class names.
  - Use [JgraphT](https://jgrapht.org/) to create a directed graph using the Map of class names to instance variables.
  - Detect cycles for each vertex in the graph using [JgraphT's](https://jgrapht.org/) [CycleDetector](https://jgrapht.org/javadoc/org/jgrapht/alg/cycle/CycleDetector.html)
  - Store graphs as PNG files using jgrapht-ext library

---

### Code Snippets

##### Collecting the Instance Variables of a Class using Java Parser
```java

  List<String> instanceVarsTypes = compilationUnit
    .findAll(FieldDeclaration.class)
    .stream()
    .map(f -> f.getVariables().get(0).getType()) // get the Variable Declarator
    .filter(v -> !v.isPrimitiveType())
    .map( v -> v.toString())				
    .filter(classNames::contains)
    .collect(Collectors.toList());

```

---

##### Add to Map < Class Name , List of Instance Variable Types >

```java

  Map<String,List<String>> classNametoInstanceVarTypesMap = new HashMap<>();		

  if(!instanceVarsTypes.isEmpty()) {
    String fileName = javaFile.getName();
    String className = fileName.substring(0, fileName.lastIndexOf('.'));
    classNametoInstanceVarTypesMap.put(className, instanceVarsTypes);
  }

```

---

##### Create Directed Graph using Map
```java

  Graph<String, DefaultEdge> graph = new DefaultDirectedGraph<>(DefaultEdge.class);		
  //add vertices
  classNametoInstanceVarTypesMap.keySet().forEach(className ->{			
    graph.addVertex(className);
  });
  //add edges
  classNametoInstanceVarTypesMap
    .forEach((className , instanceVariableTypes) -> {
        instanceVariableTypes.forEach( instVar -> {
          if (classNametoInstanceVarTypesMap.containsKey(instVar)){
            graph.addEdge(className, instVar);
          }			
      });
  });

```

---

##### Detect cycles for each vertex in the graph using JGraphT CycleDetector
```java

  CycleDetector<String, DefaultEdge> cycleDetector = new CycleDetector<String, DefaultEdge>(graph);

  cycleDetector.findCycles().forEach(vertex -> {
    AsSubgraph<String, DefaultEdge> subGraph = new AsSubgraph<>(graph, cycleDetector.findCyclesContainingVertex(vertex));
    createImage( subGraph, imageName);
  });

```
- `findCyclesContainingVertex(vertex)` returns set of vertices participating in the cycle for a vertex. Create a subgraph of the main graph using these vertices.

---

##### Create PNG Image of a graph

```java

  File imgFile = new File(imageFilePath);
  imgFile.createNewFile();

  JGraphXAdapter<String, DefaultEdge> graphAdapter = new JGraphXAdapter<String, DefaultEdge>(subGraph);

  mxIGraphLayout layout = new mxParallelEdgeLayout(graphAdapter);
  layout.execute(graphAdapter.getDefaultParent());

  BufferedImage image = mxCellRenderer.createBufferedImage(graphAdapter, null, 2, Color.WHITE, true, null);
  if(image != null) {
   ImageIO.write(image, "PNG", imgFile);
  }

```

---

#### References
- https://tomassetti.me/parsing-in-java/
- https://www.baeldung.com/jgrapht
