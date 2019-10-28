---
tags: [Java, code maintenance, source code analysis]
---

As Java projects get bigger, with several developers working on them, it may lead to code having circular
references. It is difficult to serialize objects with circular references. I have created a tool to detect circular references in this Java project : [Circular Reference Detector Gitlab Soruce Code](https://gitlab.com/nikhil_ideacrest/java-circular-reference-detector). Below are the steps taken to detect circular references in a Java project.

### Program Goal
- **Input** : Path to a Java project's source files.
- **Output** : Folder containing the image files of cyclic graphs for each class having circular references.

---

### Steps
  - Using [Java Parser](https://github.com/Javaparser/Javaparser), parse every Java file in the given source directory and collect the references of a class in a directed graph using [JgraphT](https://jgrapht.org/)
  - Detect cycles for each vertex in the graph using [JgraphT's](https://jgrapht.org/) [CycleDetector](https://jgrapht.org/Javadoc/org/jgrapht/alg/cycle/CycleDetector.html)
  - Store graphs as PNG files using jgrapht-ext library


---

### Code Snippets

##### Parse Java files in the source directory and add the class references to a graph.
``` Java
Graph<String, DefaultEdge> classReferencesGraph = new DefaultDirectedGraph<>(DefaultEdge.class);
try (Stream<Path> filesStream = Files.walk(Paths.get(srcDirectory))) {
	filesStream				
	.filter(path -> path.getFileName().toString().endsWith(".Java"))
	.forEach(path ->{
		Set<String> instanceVarTypes = getInstanceVarTypes(path.toFile());
		if( ! instanceVarTypes.isEmpty()) {						
			String className = getClassName(path.getFileName().toString());
			classReferencesGraph.addVertex(className);
			instanceVarTypes.forEach(classReferencesGraph::addVertex);
			instanceVarTypes.forEach( var -> classReferencesGraph.addEdge(className, var));
		}
	});
}
```

##### Collecting the Instance Variables of a Class using Java Parser
``` Java
CompilationUnit compilationUnit = StaticJavaParser.parse(JavaSrcFile);		
List<Node> instanceVarTypes = compilationUnit.findAll(FieldDeclaration.class)
    .stream()
    .map(f -> f.getVariables().get(0).getType())
    .filter(v -> !v.isPrimitiveType())
    .map( Object::toString)
    .collect(Collectors.toSet());
```

---

##### Create Directed Graph using Map
```Java
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

##### Detect cycles for each vertex in the graph using JGraphT CycleDetector and add the cycles to a map.
```Java
Map<String, AsSubgraph<String, DefaultEdge>> cyclesForEveryVertexMap = new HashMap<>();
CycleDetector<String, DefaultEdge> cycleDetector = new CycleDetector<>(classReferencesGraph);
cycleDetector.findCycles().forEach(v -> {
	AsSubgraph<String, DefaultEdge> subGraph = new AsSubgraph<>(classReferencesGraph,
			cycleDetector.findCyclesContainingVertex(v));
	cyclesForEveryVertexMap.put(v,subGraph);
});

```
- `findCyclesContainingVertex(vertex)` returns set of vertices participating in the cycle for a vertex. Create a subgraph of the main graph using these vertices.

---

##### Create PNG Image of a graph using jgrapht-ext library.

```Java
new File(outputDirectoryPath).mkdirs();
File imgFile = new File(outputDirectoryPath+"/graph" + imageName + ".png");
if(imgFile.createNewFile()) {
	JGraphXAdapter<String, DefaultEdge> graphAdapter = new JGraphXAdapter<>(subGraph);
	mxIGraphLayout layout = new mxCircleLayout(graphAdapter);
	layout.execute(graphAdapter.getDefaultParent());

	BufferedImage image = mxCellRenderer.createBufferedImage(graphAdapter, null, 2, Color.WHITE, true, null);
	if (image != null) {
		ImageIO.write(image, "PNG", imgFile);
	}
}

```

##### Sample Output Graph Image

![Sample output graph image.](/assets/images/detect-circular-references-in-a-java-project/graphD.png)

---

#### References
- [Circular Reference Detector Gitlab Soruce Code](https://gitlab.com/nikhil_ideacrest/java-circular-reference-detector)
- [Java parser](https://tomassetti.me/parsing-in-Java/)
- [jgrapht](https://www.baeldung.com/jgrapht)
