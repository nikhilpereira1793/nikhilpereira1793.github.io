---
tags: [java, java tools, jdeps, static code analysis]
---
In big Java projects it gets difficult to find out, visualize and filter the class, package and jar dependencies. Jdeps is a java command line tool that can be used to simplify this problem. Jdeps is a command that displays class level or package level dependencies of a jar or directory of classes.

```
//sample format of command
jdeps -verbose:class -classpath '<location of jar dependencies>' '<location of application jar or class files>'

//sample command
jdeps -verbose:class -classpath 'sample-project/target/sample-0.0.1-SNAPSHOT/lib/*'  sample-project/target/sample.jar
```
- The above command gives the dependencies at class level.
- Below is the sample output.

```
//Output
sample-1.0.0.jar -> target/lib/activation-1.1.jar
sample-1.0.0.jar -> target/lib/amazon-sqs-java-messaging-lib-
sample-1.0.0.jar -> target/lib/xstream-1.4.5.jar
   com.sample.common.mail.Attachment (sample-1.0.0.jar)
      -> java.io.Serializable                               
      -> java.lang.Object                                   
      -> java.lang.String                                   
   com.sample.common.mail.Mail (sample-1.0.0.jar)
      -> java.lang.Object                                   
      -> java.lang.String                                   
      -> java.lang.StringBuffer  
```
- We can also create visual graphs in PNG format of the dependencies using graphwiz on the dot files generated using the jdeps command.
```
jdeps -verbose:class -e '.*.slf4j.*.' -dotoutput . -classpath 'target/lib/*' target/sample-1.0.0.jar
```
- We are filtering the classes that depend on slf4j and creating a dot file in the current directory.
```
// using graphwiz convert dot file to PNG
dot -Tpng sample-1.0.0.jar.dot -o DocName.png
```
