# Spring Base Parent POM

An opinionated Maven parent POM for Spring applications in the `com.vulinh` ecosystem. It centralizes the platform versions, dependencies, and build configuration that application services are expected to share.

## Usage

Install this parent POM into the local Maven repository before a local child project can resolve it:

```shell
./mvnw install
```

On Windows PowerShell, run:

```powershell
.\mvnw.cmd install
```

For use outside the local machine, publish the parent to the Maven repository from which child projects resolve dependencies.

Use this artifact as the parent of an application POM:

```xml
<parent>
    <groupId>com.vulinh</groupId>
    <artifactId>spring-base-parent</artifactId>
    <version>2.0.5</version>
</parent>
```

Declare application-specific Spring Boot and Spring Cloud dependencies in the child POM without versions when they are managed by Spring Boot or the imported Spring Cloud BOM.

## Baseline

* Spring Boot `4.1.3`

* Java `25`

* Spring Base Commons `4.0.5`

* Log4j2 (`spring-boot-starter-log4j2`) instead of Spring Boot's default Logback logging; `spring-boot-starter-logging` is excluded from the managed `spring-boot-starter`

* Spring Boot Actuator, Data JPA, and Liquibase in every child project

* Lombok, MapStruct `1.6.3`, Hibernate static metamodel generation, OpenFeign QueryDSL `7.7`, P6Spy `2.0.1`, Commons Collections4 `4.6.0`, and Spring Base Commons

* Spring Cloud Dependencies `2025.1.5` (BOM import)

* springdoc-openapi-starter-webmvc-ui `3.2.1` for optional use

* Spring Boot Maven Plugin with opinionated build configuration

* `fmt-maven-plugin` `2.29` with google-java-format `2.0.0`, which formats Java sources on every build

* Maven Surefire and Failsafe plugins with Mockito and Byte Buddy agents for testing (required by JDK 25)

The Maven compiler configuration enables the corresponding annotation processors. Test execution is also opinionated: Surefire runs `*Test.java`, while Failsafe runs `*IT.java`; both use the configured Mockito and Byte Buddy agents.

### Test dependencies

This parent does not add `spring-boot-starter-test`. Child projects declare it themselves, without a version:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

The Surefire and Failsafe agents point at the Mockito and Byte Buddy agent JARs in the local Maven repository, using the `mockito.version` and `byte-buddy.version` properties managed by Spring Boot. These JARs are present once the test starter has been resolved.

### Code formatting

The `fmt-maven-plugin` runs its `format` goal during the build and rewrites Java sources with google-java-format. Commit the formatted result. To skip formatting for a single build, pass `-Dfmt.skip`:

```shell
./mvnw verify -Dfmt.skip
```

### Application-specific annotation processors

Do not copy the parent compiler-plugin configuration. To add an application-specific processor, configure `maven-compiler-plugin` in the child POM and append its path to the inherited list:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <annotationProcessorPaths combine.children="append">
                    <path>
                        <groupId>com.example</groupId>
                        <artifactId>example-processor</artifactId>
                        <version>1.2.3</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

The `combine.children="append"` attribute preserves the Lombok, MapStruct, Hibernate metamodel, and QueryDSL processors supplied by this parent. Omit it only when the child deliberately replaces the complete processor list.

### Test plugin customization

Surefire and Failsafe are also inherited. Append to collection settings such as `includes` or `excludes`; do not copy their configuration. For example, to let Surefire run an additional test naming pattern:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <includes combine.children="append">
                    <include>**/*Spec.java</include>
                </includes>
            </configuration>
        </plugin>
    </plugins>
</build>
```

To add JVM arguments for either test plugin, set the child project's `argLine` property rather than overriding the plugin's `argLine` configuration:

```xml
<properties>
    <argLine>-Duser.timezone=UTC</argLine>
</properties>
```

This keeps the parent-provided Mockito and Byte Buddy agents. Overriding the plugins' `argLine`, `includes`, or `excludes` without merging is a deliberate replacement of the inherited settings.

## Managed dependencies

This parent imports the Spring Cloud BOM at version `2025.1.5` and the OpenFeign QueryDSL BOM at version `7.7`, and manages `springdoc-openapi-starter-webmvc-ui` at version `3.2.1`.

### Version overrides

The version properties declared by this parent are defaults. A child POM can redefine an inherited property when it needs a different compatible version:

```xml
<properties>
    <spring-cloud.version>2025.1.6</spring-cloud.version>
</properties>
```

This applies to the properties declared in this POM: `java.version`, `spring-base-commons.version`, `spring-cloud.version`, `mapstruct.version`, `openfeign-querydsl.version`, `springdoc-openapi.version`, `p6spy-springboot.version`, `commons-collections4.version`, `fmt-maven-plugin.version`, and `google-java-format.version`. Use an override only after confirming compatibility with the selected Spring Boot and Spring Cloud releases. Dependencies managed solely by the Spring Boot parent or imported Spring Cloud BOM should generally remain versionless in child POMs.

Do not use the `querydsl.version` property for QueryDSL; Spring Boot reserves it. Override `openfeign-querydsl.version` instead.

The Spring Cloud BOM provides the version for Spring Cloud starters. For example, applications can declare the RabbitMQ binder for Spring Cloud Stream without a version:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

The imported `spring-cloud-dependencies` BOM determines that starter's compatible version. Child projects should not set a separate version unless they intentionally override this platform decision.
