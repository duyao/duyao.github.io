title: maven依赖
date: 2015-10-16 11:01:57
tags: 
	- maven 
categories: note
---

`pom.xml`
```
<dependencies>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    
    <dependency>
      <groupId>com.imooc.hi</groupId>
      <artifactId>hi</artifactId>
      <version>0.0.1-SNAPSHOT</version>
    </dependency>
    
</dependencies>

```

来源于[dependency scpoe][1]

##  依赖范围
即`scope`标签内容
There are 6 scopes available:
<!--more-->
- `compile`
This is the default scope, used if none is specified. Compile dependencies are available in all classpaths of a project. Furthermore, those dependencies are propagated(传播) to dependent projects.

- `provided`
This is much like compile, but indicates you expect the JDK or a container to provide the dependency at runtime. For example, when building a web application for the Java Enterprise Edition, you would set the dependency on the Servlet API and related Java EE APIs to scope provided because the web container provides those classes. This scope is only available on the compilation and test classpath, and is not transitive.

- `runtime`
This scope indicates that the dependency is not required for compilation, but is for execution. It is in the runtime and test classpaths, but not the compile classpath. JDBC is such kind.

- `test`
This scope indicates that the dependency is not required for normal use of the application, and is only available for the test compilation and execution phases.
system
This scope is similar to provided except that you have to provide the JAR which contains it explicitly. The artifact is always available and is not looked up in a repository.

- `import` (only available in Maven 2.0.9 or later)
This scope is only used on a dependency of type pom in the <dependencyManagement> section. It indicates that the specified POM should be replaced with the dependencies in that POM's <dependencyManag ement> section. Since they are replaced, dependencies with a scope of import do not actually participate in limiting the transitivity of a dependency.



## 依赖传递 
- Excluded dependencies 排除依赖
If project X depends on project Y, and project Y depends on project Z, the owner of project X can explicitly exclude project Z as a dependency, using the "exclusion" element.
```
<dependencies>
    <dependency>
      <groupId>com.Y</groupId>
      <artifactId>Y</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <scope>test</scope>
      
      <exclusions>
          <exclusion>
          	 <groupId>com.Z</groupId>
    		  <artifactId>Z</artifactId>
          </exclusion>
      </exclusions>
      
    </dependency>
</dependencies>
  ```
  
## 依赖冲突 Dependency mediation
- "nearest definition" 
means that the version used will be the closest one to your project in the tree of dependencies, eg. if dependencies for A, B, and C are defined as A -> B -> C -> D 2.0 and A -> E -> D 1.0, then D 1.0 will be used when building A because the path from A to D through E is shorter. You could explicitly add a dependency to D 2.0 in A to force the use of D 2.0
 
- the first declaration wins
Note that if two dependency versions are at the same depth in the dependency tree, until Maven 2.0.8 it was not defined which one would win, but since Maven 2.0.9 it's the order in the declaration that counts: the first declaration wins.

## 聚合 Project Aggregation与继承 Project Inheritance
[aggregation and inheritance][2]

###聚合要点
- Change the parent POMs packaging to the value `pom` .
- Specify in the parent POM the directories of its modules (children POMs)

###继承要点
- Elements in the POM that are merged are the following:
- dependencies
- developers and contributors
- plugin lists (including reports)
- plugin executions with matching ids
- plugin configuration
- resources



  [1]: https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Dependency_Scope
  [2]: https://maven.apache.org/guides/introduction/introduction-to-the-pom.html