







# 2Maven

## Maven介绍

Maven是一个Java项目管理和构建工具，它可以定义项目结构、项目依赖，并使用统一的方式进行自动化构建

一个Java项目需要

- 确定引入哪些依赖包
- 确定项目的目录结构：src 存放Java源码，resources存放配置文件，bin存放编译生成的.class
- 配置环境

带来的问题：琐碎且耗时



Maven的优点：

- 提供一套标准化的项目结构
- 提供一套标准化的构建流程（编译、测试、打包、发布...)
- 提供一套依赖管理机制







录`a-maven-project`是项目名，它有一个项目描述文件`pom.xml`，存放Java源码的目录是`src/main/java`，存放资源文件的目录是`src/main/resources`，存放测试源码的目录是`src/test/java`，存放测试资源的目录是`src/test/resources`，最后，所有编译、打包生成的文件都放在`target`目录里。这些就是一个Maven项目的标准目录结构。

所有的目录结构都是约定好的标准结构，不要随意修改目录结构。使用标准结构不需要做任何配置，Maven就可以正常使用。



项目描述文件`pom.xml`，内容如下：

```
<project ...>
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.itranswarp.learnjava</groupId>
	<artifactId>hello</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>
	<properties>
        ...
	</properties>
	<dependencies>
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>1.2</version>
        </dependency>
	</dependencies>
</project>
```

`groupId`类似于Java的包名，**通常是公司或组织名称**，`artifactId`类似于Java的类名，通常是项目名称，再加上`version`，一个Maven工程就是由`groupId`，`artifactId`和`version`作为唯一标识。我们在引用其他第三方库的时候，也是通过这3个变量确定。例如，依赖`commons-logging`：

```
<dependency>
    <groupId>commons-logging</groupId>
    <artifactId>commons-logging</artifactId>
    <version>1.2</version>
</dependency>
```

使用`<dependency>`声明一个依赖后，Maven就会自动下载这个依赖包并把它放到classpath中。



在 Maven 中，groupId 和 artifactId 代表了 Maven 项目的核心信息。

- groupId（组标识）：这个标识通常是组织或公司的唯一标识，通常使用反转的域名，比如 `com.example`。groupId 用于标识同一个组织或公司的项目，可以将多个项目组织在一个父工程下。
- artifactId（项目标识）：这个标识是指项目的名称或者模块的名称，例如 `my-project` 或 `my-library`。artifactId 用于标识一个具体的项目或者模块，可以作为该项目或模块所生成的 jar 包的名称的一部分。

通常情况下，groupId 和 artifactId 一起构成 Maven 项目唯一的标识符。在 Maven 中，通过这些标识符来唯一确定项目的依赖关系和版本号，以确保项目构建和部署的正确性。





## 依赖管理

Maven解决了依赖管理问题，例如，我们的项目依赖`abc`这个jar包，而`abc`又依赖`xyz`这个jar包：

```ascii
┌──────────────┐
│Sample Project│
└──────────────┘
        │
        ▼
┌──────────────┐
│     abc      │
└──────────────┘
        │
        ▼
┌──────────────┐
│     xyz      │
└──────────────┘
```

当我们声明了`abc`的依赖时，Maven自动把`abc`和`xyz`都加入了我们的项目依赖，不需要我们自己去研究`abc`是否需要依赖`xyz`。



- 依赖关系

  Maven定义了几种依赖关系，分别是`compile`、`test`、`runtime`和`provided`：

  | scope    | 说明                                          | 示例            |
  | :------- | :-------------------------------------------- | :-------------- |
  | compile  | 编译时需要用到该jar包（默认）                 | commons-logging |
  | test     | 编译Test时需要用到该jar包                     | junit           |
  | runtime  | 编译时不需要，但运行时需要用到                | mysql           |
  | provided | 编译时需要用到，但运行时由JDK或某个服务器提供 | servlet-api     |

  其中，默认的`compile`是最常用的，Maven会把这种类型的依赖直接放入classpath。

  `test`依赖表示仅在测试时使用，正常运行时并不需要。最常用的`test`依赖就是JUnit：

  ```
  <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>5.3.2</version>
      <scope>test</scope>
  </dependency>
  ```

  `runtime`依赖表示编译时不需要，但运行时需要。最典型的`runtime`依赖是JDBC驱动，例如MySQL驱动：

  ```
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.48</version>
      <scope>runtime</scope>
  </dependency>
  ```

  `provided`依赖表示编译时需要，但运行时不需要。最典型的`provided`依赖是Servlet API，编译的时候需要，但是运行时，Servlet服务器内置了相关的jar，所以运行期不需要：

  ```
  <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.0</version>
      <scope>provided</scope>
  </dependency>
  ```

  最后一个问题是，Maven如何知道从何处下载所需的依赖？也就是相关的jar包？答案是**Maven维护了一个中央仓库**（[repo1.maven.org](https://repo1.maven.org/)），所有第三方库将自身的jar以及相关信息上传至中央仓库，Maven就可以从中央仓库把所需依赖下载到本地。

  Maven并不会每次都从中央仓库下载jar包。一个jar包一旦被下载过，就会被Maven自动缓存在本地目录（用户主目录的`.m2`目录），所以，除了第一次编译时因为下载需要时间会比较慢，后续过程因为有本地缓存，并不会重复下载相同的jar包。

- 唯一ID

  对于某个依赖，Maven只需要3个变量即可唯一确定某个jar包：

  - groupId：属于组织的名称，类似Java的包名；
  - artifactId：该jar包自身的名称，类似Java的类名；
  - version：该jar包的版本。

  通过上述3个变量，即可唯一确定某个jar包。Maven通过对jar包进行PGP签名确保任何一个jar包一经发布就无法修改。修改已发布jar包的唯一方法是发布一个新版本。****

  注：只有以`-SNAPSHOT`结尾的版本号会被Maven视为开发版本，开发版本每次都会重复下载，这种SNAPSHOT版本只能用于内部私有的Maven repo，公开发布的版本不允许出现SNAPSHOT。

- Maven镜像

  除了可以从Maven的中央仓库下载外，还可以从Maven的镜像仓库下载。如果访问Maven的中央仓库非常慢，我们可以选择一个速度较快的Maven的镜像仓库。Maven镜像仓库定期从中央仓库同步：

  ```ascii
             slow    ┌───────────────────┐
      ┌─────────────▶│Maven Central Repo.│
      │              └───────────────────┘
      │                        │
      │                        │sync
      │                        ▼
  ┌───────┐  fast    ┌───────────────────┐
  │ User  │─────────▶│Maven Mirror Repo. │
  └───────┘          └───────────────────┘
  ```

  中国区用户可以使用阿里云提供的Maven镜像仓库。使用Maven镜像仓库需要一个配置，在用户主目录下进入`.m2`目录，创建一个`settings.xml`配置文件，内容如下：

  ```
  <settings>
      <mirrors>
          <mirror>
              <id>aliyun</id>
              <name>aliyun</name>
              <mirrorOf>central</mirrorOf>
              <!-- 国内推荐阿里云的Maven镜像 -->
              <url>https://maven.aliyun.com/repository/central</url>
          </mirror>
      </mirrors>
  </settings>
  ```

- 搜索第三方组件

  如果我们要引用一个第三方组件，比如`okhttp`，可以通过[search.maven.org](https://search.maven.org/)搜索关键字，找到对应的组件后直接复制

- 命令行编译

  进入到`pom.xml`所在目录，输入以下命令：

  ```
  $ mvn clean package
  ```



## 构建流程

- Lifecycle和Phase

  Maven的生命周期由一系列阶段（phase）构成，以内置的生命周期`default`为例，它包含以下phase：

  - validate
  - initialize
  - generate-sources
  - process-sources
  - generate-resources
  - process-resources
  - compile
  - process-classes
  - generate-test-sources
  - process-test-sources
  - generate-test-resources
  - process-test-resources
  - test-compile
  - process-test-classes
  - test
  - prepare-package
  - package
  - pre-integration-test
  - integration-test
  - post-integration-test
  - verify
  - install
  - deploy

  如果运行`mvn package`，Maven就会执行`default`生命周期，它会从开始一直运行到`package`这个phase为止

  如果运行`mvn compile`，Maven也会执行`default`生命周期，但这次它只会运行到`compile`

  Maven另一个常用的生命周期是`clean`，它会执行3个phase：

  - pre-clean
  - clean （注意这个clean不是lifecycle而是phase）
  - post-clean
  
  所以，我们使用`mvn`这个命令时，后面的参数是phase，Maven自动根据生命周期运行到指定的phase。
  
  更复杂的例子是指定多个phase，例如，运行`mvn clean package`，Maven先执行`clean`生命周期并运行到`clean`这个phase，然后执行`default`生命周期并运行到`package`这个phase，实际执行的phase如下：
  
  - pre-clean
  - clean （注意这个clean是phase）
  - validate
  - ...
  - package
  
  在实际开发过程中，经常使用的命令有：
  
  `mvn clean`：清理所有生成的class和jar；
  
  `mvn clean compile`：先清理，再执行到`compile`；
  
  `mvn clean test`：先清理，再执行到`test`，因为执行`test`前必须执行`compile`，所以这里不必指定`compile`；
  
  `mvn clean package`：先清理，再执行到`package`。

- Goal

  执行一个phase又会触发一个或多个goal：

  | 执行的Phase | 对应执行的Goal                     |
  | :---------- | :--------------------------------- |
  | compile     | compiler:compile                   |
  | test        | compiler:testCompile surefire:test |

  goal的命名总是`abc:xyz`这种形式。

总结：

- lifecycle相当于Java的package，它包含一个或多个phase；
- phase相当于Java的class，它包含一个或多个goal；
- goal相当于class的method，它其实才是真正干活的。

大多数情况，我们只要指定phase，就默认执行这些phase默认绑定的goal，只有少数情况，我们可以直接指定运行一个goal，例如，启动Tomcat服务器：

```
mvn tomcat:run
```



## 使用插件

使用Maven构建项目就是执行lifecycle，执行到指定的phase为止。每个phase会执行自己默认的一个或多个goal。goal是最小任务单元。

我们以`compile`这个phase为例，如果执行：

```
mvn compile
```

Maven将执行`compile`这个phase，这个phase会调用`compiler`插件执行关联的`compiler:compile`这个goal。

实际上，执行每个phase，都是通过某个插件（plugin）来执行的，Maven本身其实并不知道如何执行`compile`，它只是负责找到对应的`compiler`插件，然后执行默认的`compiler:compile`这个goal来完成编译。

Maven已经内置了一些常用的标准插件：

| 插件名称 | 对应执行的phase |
| :------- | :-------------- |
| clean    | clean           |
| compiler | compile         |
| surefire | test            |
| jar      | package         |

如果标准插件无法满足需求，我们还可以使用自定义插件。使用自定义插件的时候，需要声明。例如，使用`maven-shade-plugin`可以创建一个可执行的jar，要使用这个插件，需要在`pom.xml`中声明它：

```
<project>
    ...
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-shade-plugin</artifactId>
                <version>3.2.1</version>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>shade</goal>
						</goals>
						<configuration>
                            ...
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```

## 模块管理

在软件开发中，把一个大项目分拆为多个模块是降低软件复杂度的有效方法：

```ascii
                        ┌ ─ ─ ─ ─ ─ ─ ┐
                          ┌─────────┐
                        │ │Module A │ │
                          └─────────┘
┌──────────────┐ split  │ ┌─────────┐ │
│Single Project│───────▶  │Module B │
└──────────────┘        │ └─────────┘ │
                          ┌─────────┐
                        │ │Module C │ │
                          └─────────┘
                        └ ─ ─ ─ ─ ─ ─ ┘
```

对maven工程来说，原来是一个大项目：

```ascii
single-project
├── pom.xml
└── src
```

现在可以分拆成3个模块：

```ascii
mutiple-project
├── module-a
│   ├── pom.xml
│   └── src
├── module-b
│   ├── pom.xml
│   └── src
└── module-c
    ├── pom.xml
    └── src
```

Maven可以有效地管理多个模块，我们只需要把每个模块当作一个独立的Maven项目，它们有各自独立的`pom.xml`

- 中央仓库

  其实我们使用的大多数第三方模块都是这个用法，例如，我们使用commons logging、log4j这些第三方模块，就是第三方模块的开发者自己把编译好的jar包发布到Maven的中央仓库中。

- 私有仓库

  私有仓库是指公司内部如果不希望把源码和jar包放到公网上，那么可以搭建私有仓库。私有仓库总是在公司内部使用，它只需要在本地的`~/.m2/settings.xml`中配置好，使用方式和中央仓位没有任何区别。

- 本地仓库

  本地仓库是指把本地开发的项目“发布”在本地，这样其他项目可以通过本地仓库引用它。但是我们不推荐把自己的模块安装到Maven的本地仓库，因为每次修改某个模块的源码，都需要重新安装，非常容易出现版本不一致的情况。更好的方法是使用模块化编译，在编译的时候，告诉Maven几个模块之间存在依赖关系，需要一块编译，Maven就会自动按依赖顺序编译这些模块。

## 发布Artifact

当我们使用`commons-logging`这些第三方开源库的时候，我们实际上是通过Maven自动下载它的jar包，并根据其`pom.xml`解析依赖，自动把相关依赖包都下载后加入到classpath。

如果我们把自己的开源库放到Maven的repo中，那么，别人只需按标准引用`groupId:artifactId:version`，即可自动下载jar包以及相关依赖。因此，本节我们介绍如何发布一个库到Maven的repo中。

把自己的库发布到Maven的repo中有好几种方法，我们介绍3种最常用的[方法](https://www.liaoxuefeng.com/wiki/1252599548343744/1347981037010977)。

- 可以发布到本地，然后推送到远程Git库，由静态服务器提供基于网页的repo服务，使用方必须声明repo地址；
- 可以发布到[central.sonatype.org](https://central.sonatype.org/)，并自动同步到Maven中央仓库，需要前期申请账号以及本地配置；
- 可以发布到GitHub Packages作为私有仓库使用，必须提供Token以及正确的权限才能发布和使用。

