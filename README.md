Sample plugin that defines SonarQube custom rules for Cucumber Gherkin feature files
====================================================================================

[![Build Status](https://api.travis-ci.org/racodond/sonar-gherkin-custom-rules-plugin.svg?branch=master)](https://travis-ci.org/racodond/sonar-gherkin-custom-rules-plugin)
[![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/cwah7epa41uqc9we/branch/master?svg=true)](https://ci.appveyor.com/project/racodond/sonar-gherkin-custom-rules-plugin/branch/master)
[![Quality Gate](https://sonarqube.com/api/badges/gate?key=com.racodond.plugin.gherkin:sonar-gherkin-custom-rules-plugin)](https://sonarqube.com/overview?id=com.racodond.sonarqube.plugin.gherkin%3Asonar-gherkin-custom-rules-plugin)

## Description
The [SonarQube Gherkin plugin](https://github.com/racodond/sonar-gherkin-plugin) can be enhanced by writing custom rules through a plugin using SonarQube Gherkin API.
This sample plugin is designed to help you get started writing your own plugin and custom rules.

## Usage
1. [Download and install](http://docs.sonarqube.org/display/SONAR/Setup+and+Upgrade) SonarQube 5.6 or greater
1. Install the Gherkin plugin (1.0 or greater) either by a [direct download](https://github.com/racodond/sonar-gherkin-plugin/releases) or through the [Update Center](http://docs.sonarqube.org/display/SONAR/Update+Center).
1. Install this sample plugin by a [direct download](https://github.com/racodond/sonar-gherkin-custom-rules-plugin/releases)
1. Start SonarQube
1. [Activate some of the custom rules](http://docs.sonarqube.org/display/SONAR/Configuring+Rules) implemented in this sample plugin ("Forbidden tag" for example).
1. [Install your favorite analyzer](http://docs.sonarqube.org/display/SONAR/Analyzing+Source+Code#AnalyzingSourceCode-RunningAnalysis) (SonarQube Scanner, Maven, Ant, etc.) and analyze your code. Note that Java 8 is required to run an analysis.
1. Browse the issues through the web interface 

## Writing Custom Rules

### Creating a SonarQube Plugin
* Create a [standard SonarQube plugin](http://docs.sonarqube.org/display/DEV/Build+Plugin) from scratch or start from this sample plugin
* Attach this plugin to the SonarQube Gherkin plugin through the [POM](pom.xml):
  * Add the [dependency](pom.xml#L71) to the Gherkin plugin
  * Add the following property to the [`sonar-packaging-maven-plugin` configuration](pom.xml#L105):
 ```
 <basePlugin>gherkin</basePlugin>
 ```
* Implement the following extension points:
  * [Plugin](http://javadocs.sonarsource.org/latest/apidocs/index.html?org/sonar/api/Plugin.html) as in [`MyGherkinCustomRulesPlugin.java`](src/main/java/org/sonar/gherkin/MyGherkinCustomRulesPlugin.java)
  * [RulesDefinition](http://javadocs.sonarsource.org/latest/apidocs/index.html?org/sonar/api/server/rule/RulesDefinition.html) as in [`MyGherkinCustomRulesDefinition.java`](src/main/java/org/sonar/gherkin/MyGherkinCustomRulesDefinition.java)
* Declare the [`RulesDefinition` implementation as an extension in the `Plugin` extension point](src/main/java/org/sonar/gherkin/MyGherkinCustomRulesPlugin.java#L34).

### Implementing a Rule
* Create a class to define the implementation of a rule. It should:
  * Either extend [`SubscriptionVisitorCheck`](https://github.com/racodond/sonar-gherkin-plugin/blob/master/gherkin-frontend/src/main/java/org/sonar/plugins/gherkin/api/visitors/SubscriptionVisitorCheck.java) or [`DoubleDispatchVisitorCheck`](https://github.com/racodond/sonar-gherkin-plugin/blob/master/gherkin-frontend/src/main/java/org/sonar/plugins/gherkin/api/visitors/DoubleDispatchVisitorCheck.java).
  * Define the [rule's attributes](src/main/java/org/sonar/gherkin/checks/ForbiddenTagCheck.java#L32): key, name, priority, etc.
* Declare this class in the [class implementing `RulesDefinition`](src/main/java/org/sonar/gherkin/MyGherkinCustomRulesDefinition.java#L51)

There are two different ways to browse the AST:

#### Using DoubleDispatchVisitorCheck
To explore part of the AST, override a method from [`DoubleDispactchVisitor`](https://github.com/racodond/sonar-gherkin-plugin/blob/master/gherkin-frontend/src/main/java/org/sonar/plugins/gherkin/api/visitors/DoubleDispatchVisitor.java).
For instance, if you want to explore tag nodes, override [`DoubleDispactchVisitor#visitTag`](https://github.com/racodond/sonar-gherkin-plugin/blob/master/gherkin-frontend/src/main/java/org/sonar/plugins/gherkin/api/visitors/DoubleDispatchVisitor.java#L105). This method is called each time a tag node is encountered in the AST.
Note: When overriding a visit method, you must call the super method in order to allow the visitor to visit the children of the node.
See [`ForbiddenTagCheck`](src/main/java/org/sonar/gherkin/checks/ForbiddenTagCheck.java) for example.


#### Using SubscriptionVisitorCheck
To explore part of the AST, override [`SubscriptionVisitor#nodesToVisit`](https://github.com/racodond/sonar-gherkin-plugin/blob/master/gherkin-frontend/src/main/java/org/sonar/plugins/gherkin/api/visitors/SubscriptionVisitor.java#L35) by returning the list of [`Tree#Kind`](https://github.com/racodond/sonar-gherkin-plugin/blob/master/gherkin-frontend/src/main/java/org/sonar/plugins/gherkin/api/tree/Tree.java#L31) nodes you want to visit.
For instance, if you want to explore tag nodes the method should return a list containing [`Tree#Kind#TAG`](https://github.com/racodond/sonar-gherkin-plugin/blob/master/gherkin-frontend/src/main/java/org/sonar/plugins/gherkin/api/tree/Tree.java#L42).
See [`ForbiddenNameContentCheck`](src/main/java/org/sonar/gherkin/checks/ForbiddenNameContentCheck.java) for example.

#### Creating Issues
Precise issue or file issue or line issue can be created by calling the related method in [Issues](https://github.com/racodond/sonar-gherkin-plugin/blob/master/gherkin-frontend/src/main/java/org/sonar/gherkin/visitors/Issues.java).

#### Testing
Testing is made easy by the [GherkinCheckVerifier](https://github.com/racodond/sonar-gherkin-plugin/blob/master/gherkin-checks-testkit/src/main/java/org/sonar/gherkin/checks/verifier/GherkinCheckVerifier.java).
There are two ways to assert that an issue should be raised:
* Through comments directly in the .feature test file
* Or using assertions in the check class test

Examples of coding rule implementation and testing can be found in the Gherkin plugin [`gherkin-checks` module](https://github.com/racodond/sonar-gherkin-plugin/tree/master/gherkin-checks/src/main/java/org/sonar/gherkin/checks).
