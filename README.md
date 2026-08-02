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
    <version>1.0.0</version>
</parent>
```

Declare application-specific Spring Boot and Spring Cloud dependencies in the child POM without versions when they are managed by Spring Boot or the imported Spring Cloud BOM.

## Baseline

* Spring Boot `3.5.16`

* Java `25`

* Spring Base Commons `3.1.0`

* Log4j2 `2.26.1` instead of Spring Boot's default Logback logging

* Spring Boot Actuator, Data JPA, Liquibase and `spring-boot-starter-test` in every child project

* Lombok, MapStruct `1.6.3`, Hibernate static metamodel generation, OpenFeign QueryDSL `7.4.0`, P6Spy `1.12.1`, and Spring Base Commons

* Spring Cloud Dependencies `2025.0.3` (BOM import)

* Commons Lang3 `3.20.0` and springdoc-openapi-starter-webmvc-ui `2.8.17` for optional use

* Spring Boot Maven Plugin with opinionated build configuration

* Maven Surefire and Failsafe plugins with Mockito and Byte Buddy agents for testing (required by JDK 25)

### CVE fixes

This parent manages versions for libraries that address known security vulnerabilities shared across all projects:

* Commons Lang3 `3.20.0`

* Log4j2 `2.26.1`

The Maven compiler configuration enables the corresponding annotation processors. Test execution is also opinionated: Surefire runs `*Test.java`, while Failsafe runs `*IT.java`; both use the configured Mockito and Byte Buddy agents.

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

This parent imports the Spring Cloud BOM at version `2025.0.3`, the OpenFeign QueryDSL BOM at version `7.4.0`, and manages `springdoc-openapi-starter-webmvc-ui` at version `2.8.17`.

### Version overrides

The version properties declared by this parent are defaults. A child POM can redefine an inherited property when it needs a different compatible version:

```xml
<properties>
    <spring-cloud.version>2025.0.4</spring-cloud.version>
    <mapstruct.version>1.6.3</mapstruct.version>
</properties>
```

This applies to the properties declared in this POM, including `java.version`, `spring-base-commons.version`, `spring-cloud.version`, `mapstruct.version`, `openfeign-querydsl.version`, `springdoc-openapi.version`, `p6spy-springboot.version`, `log4j2.version`, and `commons-lang3.version`. Use an override only after confirming compatibility with the selected Spring Boot and Spring Cloud releases. Dependencies managed solely by the Spring Boot parent or imported Spring Cloud BOM should generally remain versionless in child POMs.

The Spring Cloud BOM provides the version for Spring Cloud starters. For example, applications can declare the RabbitMQ-backed Spring Cloud Bus starter without a version:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```

The imported `spring-cloud-dependencies` BOM determines that starter's compatible version. Child projects should not set a separate version unless they intentionally override this platform decision.
