Technische Dokumentationen sind ein komplizierteres Thema als manchmal gedacht: 
Welches Format, welcher Inhalt, wo und von wem soll die Dokumentation gepflegt werden? 
Insbesondere _die Pflege_ der Dokumentation ist ein Dauerthema. 
Mit _[AsciiDoc](//asciidoc.org/)_ und dem _[asciidoctor-maven-plugin](//github.com/asciidoctor/asciidoctor-maven-plugin)_
lassen sich Dokumentationen in einem flexiblen Dokument pflegen, das Code nah 
als Modul im Projekt gepflegt und mit GIT versioniert werden kann.
<!--more-->

# Wer?

In den meisten Projekten kann und sollte die technische Dokumentation von den Entwicklern gepflegt werden,
denn nur sie ändern und erweitern Konfigurationen oder Aufbau der Software anhand von Anforderungen.

# Wo?

Bei der Pflege in einem Wiki oder einem anderen System entsteht für den Entwickler ein
Systembruch. Er muss Konfigurationsdateien oder Beispiel-Code an zwei Stellen pflegen: 
Im Code-Modul und im Wiki (in der Dokumentation). Manche Wikis erleichtern die Arbeit, indem sie
es ermöglichen, Text aus anderen Quellen, wie einem GIT Repository, einzubinden.

Mit AsciiDoc kann die Dokumentation in einem Dokument gepflegt werden, 
das in einem zusätzlichen Maven-Modul abgelegt werden kann. 
Von dort aus können Dateien aus den anderen Code-Modulen eingebunden werden.
So kann die Dokumentation direkt im Code-Modul mit der IDE gepflegt werden.

Der folgende Schnipsel erzeugt einen Abschnitt mit Überschrift Größe 2 und dem als XML formatierten Inhalt 
der Datei `log4j2.xml` aus dem Modul `myproject/mainmodule`:  

```asciidoc
=== log4j2.xml

`/WEB-INF/classes/log4j2.xml` ist die Konfiguration des Frameworks log4j2footnote:[https://logging.apache.org/log4j/2.x/], 
welches das Logging der Anwendung übernimmt.
Hier lassen sich Einstellungen des Logverhaltens der Anwendung vornehmen.

[source,xml]
----
include::../../resources/log4j2.xml[]
----
```

![xml-einbindung-log4j2.png](/images/2008/xml-einbindung-log4j2.png "Generierte Seite im PDF für die XML-Einbindung.")


Bei richtiger Organisation kann man Codeschnipsel, Beispielkonfigurationen verschiedener Systeme und weitere
Ressourcen aus den Code-Modulen einbinden.


# Welche Formate?

In vielen Projekten möchte man die Dokumentation in verschiedenen Formaten ausgeben: 
HTML für den Zugriff über den Browser und als dynamisch verlinkte Version,
PDF als Druck-freundliches Format und als statische Version für den Download,
und vielleicht noch [DocBook](//de.wikipedia.org/wiki/DocBook) als maschinenlesbares XML-Format.

Das alles und noch viel mehr Formate können mithilfe von AsciiDoc und dem asciidoctor-maven-plugin 
erzeugt werden.


# Wie?

Im Folgenden wird die Generierung der AsciiDoc-Dokumentation über die Konfiguration der Plugins
im `pluginManagement`-Bereich der pom.xml beschrieben.

Das Plugin `buildnumber-maven-plugin` erzeugt eine Property mit dem aktuellen 
Datum und der Uhrzeit, die als Build-Timestamp in der Dokumentation verwendet werden kann.
Sie wird in der Variable `revdate` gespeichert.

```xml
<properties>
    ...
    <buildnumber-maven-plugin.version>1.4</buildnumber-maven-plugin.version>
    ...
</properties>
```

```xml
<pluginManagement>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>buildnumber-maven-plugin</artifactId>
            <version>${buildnumber-maven-plugin.version}</version>
            <executions>
                <execution>
                    <id>buildCurrentDate</id>
                    <goals>
                        <goal>create-timestamp</goal>
                    </goals>
                    <phase>generate-resources</phase>
                    <configuration>
                        <timestampFormat>yyyy-MM-dd HH:mm:ss</timestampFormat>
                        <timestampPropertyName>revdate</timestampPropertyName>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        ...
    </plugins>
</pluginManagement>
```

Dem folgt das Plugin `asciidoctor-maven-plugin`, welches die eigentliche Arbeit erledigt.

```xml
<properties>
    ...
    <!-- Hieraus werden die Dateinamen der generierten Dokumentation erstellt, z. B. "Technische Dokumentation.pdf" -->
    <generatedDocumentName>${project.artifactId}-Dokumentation-${project.version}</generatedDocumentName>
    ...
    <asciidoctor-maven-plugin.version>1.5.7.1</asciidoctor-maven-plugin.version>
    <asciidoctorj.version>1.6.2</asciidoctorj.version>
    <asciidoctorj-diagram.version>1.5.12</asciidoctorj-diagram.version>
    <asciidoctorj-pdf.version>1.5.0-beta.2</asciidoctorj-pdf.version>
    ...
</properties>
```

```xml
<pluginManagement>
    <plugins>
        ...                
        <plugin>
            <groupId>org.asciidoctor</groupId>
            <artifactId>asciidoctor-maven-plugin</artifactId>
            <version>${asciidoctor-maven-plugin.version}</version>
            <configuration>
                <attributes>
                    <docinfo/>
                    <lang>de</lang>
                </attributes>
                <requires>
                    <require>asciidoctor-diagram</require>
                    <require>uri</require>
                </requires>
            </configuration>
            <executions>
                <!-- PDF Format erzeugen -->
                <execution>
                    <id>documentation-output-pdf</id>
                    <phase>prepare-package</phase>
                    <goals>
                        <goal>process-asciidoc</goal>
                    </goals>
                    <configuration>
                        <sourceDocumentName>tech-doku.adoc</sourceDocumentName>
                        <outputFile>${project.build.directory}/generated-docs/${generatedDocumentName}.pdf</outputFile>
                        <sourceHighlighter>coderay</sourceHighlighter>
                        <backend>pdf</backend>
                        <attributes>
                            <toc/>
                            <linkcss>false</linkcss>
                            <pdf-stylesdir>${project.basedir}/src/main/asciidoc/theme</pdf-stylesdir>
                            <pdf-fontsdir>${project.basedir}/src/main/asciidoc/theme/fonts</pdf-fontsdir>
                            <pdf-style>custom</pdf-style>
                            <imagesDir>${project.build.directory}/generated-docs/images</imagesDir>
                            <doctype>book</doctype>
                            <icons>font</icons>
                            <pagenums/>
                            <sectnums>true</sectnums>
                            <revnumber>${project.version}</revnumber>
                            <revdate>${revdate}</revdate>
                            <java_version>${maven.compiler.target}</java_version>
                            <!-- weitere variablen können übergeben werden -->
                            <idprefix/>
                            <idseparator>-</idseparator>
                            <autofit-option>true</autofit-option>
                        </attributes>
                    </configuration>
                </execution>

                <!-- HTML Format erzeugen -->
                <execution>
                    <id>documentation-output-html</id>
                    <phase>prepare-package</phase>
                    <goals>
                        <goal>process-asciidoc</goal>
                    </goals>
                    <configuration>
                        <sourceDocumentName>tech-doku.adoc</sourceDocumentName>
                        <outputFile>${project.build.directory}/generated-docs/${generatedDocumentName}.html</outputFile>
                        <sourceHighlighter>coderay</sourceHighlighter>
                        <backend>html</backend>
                        <outputDirectory>${project.build.directory}/generated-docs</outputDirectory>
                        <attributes>
                            <toc/>
                            <linkcss>false</linkcss>
                        </attributes>
                    </configuration>
                </execution>

                <!-- DOCBOOK Format erzeugen -->
                <execution>
                    <id>documentation-output-docbook</id>
                    <phase>prepare-package</phase>
                    <goals>
                        <goal>process-asciidoc</goal>
                    </goals>
                    <configuration>
                        <sourceDocumentName>tech-doku.adoc</sourceDocumentName>
                        <outputFile>${project.build.directory}/generated-docs/${generatedDocumentName}.xml</outputFile>
                        <backend>docbook</backend>
                        <doctype>book</doctype>
                    </configuration>
                </execution>
            </executions>
            <dependencies>
                <dependency>
                    <groupId>org.asciidoctor</groupId>
                    <artifactId>asciidoctorj-diagram</artifactId>
                    <version>${asciidoctorj-diagram.version}</version>
                </dependency>
                <dependency>
                    <groupId>org.asciidoctor</groupId>
                    <artifactId>asciidoctorj-pdf</artifactId>
                    <version>${asciidoctorj-pdf.version}</version>
                </dependency>
                <dependency>
                    <groupId>org.asciidoctor</groupId>
                    <artifactId>asciidoctorj</artifactId>
                    <version>${asciidoctorj.version}</version>
                </dependency>
            </dependencies>
        </plugin>
        ...
    </plugins>
</pluginManagement>
```


# Erweiterungen

AsciiDoc bringt von Haus aus primär die Möglichkeit mit, Texte zu formatieren und 
zu strukturieren und Grafiken einzubinden.

In Softwareprojekten sind aber noch einige Erweiterungen interessant:
UML-Diagramme und Webservice-Schnittstellen-Beschreibungen.


## PlantUML

Möchte man z. B. UML-Diagramme zur Dokumentation verwenden, ist man erst einmal darauf angewiesen,
die Diagramme in einem externen Programm zu erstellen und als PNG oder SVG einzubinden.

Mithilfe von [PlantUML](//plantuml.com/) lassen sich UML-Diagramme in AsciiDoc-Dokumente einbinden.
Dabei werden die UML-Diagramme in einem Textformat beschrieben und von PlantUML in Grafiken umgewandelt.
Dies erspart die Verwendung externer Tools und die Pflege von Grafikdateien.

```asciidoc
[plantuml,"some-diagramm",png, title="Ein Aktivitätsdiagramm"]
----
@startuml
(*) --> if "Some Test" then

  -->[true] "activity 1"

  if "" then
    -> "activity 3" as a3
  else
    if "Other test" then
      -left-> "activity 5"
    else
      --> "activity 6"
    endif
  endif

else

  ->[false] "activity 2"

endif

a3 --> if "last test" then
  --> "activity 7"
else
  -> "activity 8"
endif
@enduml
----
```

![plant-uml-aktivitaets-diagramm.png](/images/2008/plantuml-aktivitaetsdiagramm.png "Generierter Abschnitt im PDF für das UML-Diagramm.")

Damit das funktioniert, benötigt PlantUML eine Installation von [Graphviz](//graphviz.org/download/),
welche es für verschiedene Systeme gibt, darunter Windows und Linux (inkl. MacOS).

Damit auf Systemen, die kein Graphviz installiert haben, das Dokumentationsmodul
nicht fehlschlägt, kann man Profile definieren, welche erst dann aktiv werden
und die Generierung der Dokumentation mit AsciiDoc und PlantUML ausführen, wenn Graphviz auch installiert
ist:

```xml
<profiles>
    <profile>
        <!-- linux platforms -->
        <id>gendocLin</id>
        <activation>
            <file>
                <exists>/usr/bin/dot</exists>
            </file>
        </activation>
        <!-- Build Workflow -->
        <build>
            <plugins>
                <plugin>
                    <groupId>org.asciidoctor</groupId>
                    <artifactId>asciidoctor-maven-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-dependency-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-assembly-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <configuration>
                        <skip>${skipSwaggerAdocGeneration}</skip>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <skip>${skipSwaggerAdocGeneration}</skip>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-resources-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>io.github.swagger2markup</groupId>
                    <artifactId>swagger2markup-maven-plugin</artifactId>
                    <configuration>
                        <skip>${skipSwaggerAdocGeneration}</skip>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>

    <profile>
        <!-- windows platforms -->
        <id>gendocWin</id>
        <activation>
            <file>
                <exists>${user.home}/graphviz/dot.exe</exists>
            </file>
        </activation>
        <!-- Build Workflow -->
        <build>
            <plugins>
                <plugin>
                    <groupId>org.asciidoctor</groupId>
                    <artifactId>asciidoctor-maven-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-dependency-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-assembly-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <configuration>
                        <skip>${skipSwaggerAdocGeneration}</skip>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-surefire-plugin</artifactId>
                    <configuration>
                        <skip>${skipSwaggerAdocGeneration}</skip>
                    </configuration>
                </plugin>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-resources-plugin</artifactId>
                </plugin>
                <plugin>
                    <groupId>io.github.swagger2markup</groupId>
                    <artifactId>swagger2markup-maven-plugin</artifactId>
                    <configuration>
                        <skip>${skipSwaggerAdocGeneration}</skip>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

## Swagger

Die meisten Anwendungen kommen um eine eigene Webservice-Schnittstelle nicht herum.
Diese wird dann vorzugsweise als JSON und REST-Schnittstelle umgesetzt. 
[Swagger](//swagger.io) bietet hier eine große Sammlung an Werkzeugen zum 
Erstellen und Dokumentieren solcher Webservices. 

Solche Schnittstellenbeschreibungen händisch in der Dokumentation aktuell zu halten ist mühselig.

Glücklicherweise gibt es auch hier eine Lösung:

Üblicherweise lässt sich aus der Anwendung heraus
eine JSON-Schnittstellenbeschreibung generieren, die dann als `swagger.json` zur Verfügung steht.
Man kann diese Datei sogar während der Ausführung der Maven-Prozesse aus einem 
laufenden Webservice-Endpunkt mit Swagger extrahieren lassen.

Mit dem [swagger2markup-maven-plugin](//github.com/Swagger2Markup/swagger2markup-maven-plugin) 
lassen sich diese Schnittstellenbeschreibungen in AsciiDoc umwandeln und von dort aus in
die technische Dokumentation aufnehmen.

Auszug aus der `swagger.json` für das Demo-Projekt [petstore](//petstore.swagger.io):
```json
{
  "swagger": "2.0",
  "info": {
    ...
  },
  "host": "petstore.swagger.io",
  "basePath": "/v2",
  ...
  "schemes": ["http"],
  "paths": {
      ...
      "post": {
        "tags": ["pet"],
        "summary": "Updates a pet in the store with form data",
        "description": "",
        "operationId": "updatePetWithForm",
        "consumes": ["application/x-www-form-urlencoded"],
        "produces": ["application/xml", "application/json"],
        "parameters": [
          {
            "name": "petId",
            "in": "path",
            "description": "ID of pet that needs to be updated",
            "required": true,
            "type": "integer",
            "format": "int64"
          },
          {
            "name": "name",
            "in": "formData",
            "description": "Updated name of the pet",
            "required": false,
            "type": "string"
          },
          {
            "name": "status",
            "in": "formData",
            "description": "Updated status of the pet",
            "required": false,
            "type": "string"
          }
        ],
        "responses": {
          "405": {
            "description": "Invalid input"
          }
        },
        "security": [
          {
            "petstore_auth": ["write:pets", "read:pets"]
          }
        ]
      },
      ...
    }
  },
  "definitions": {
    ...              
    "Pet": {
      "type": "object",
      "required": ["name", "photoUrls"],
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64"
        },
        "category": {
          "$ref": "#/definitions/Category"
        },
        "name": {
          "type": "string",
          "example": "doggie"
        },
        "photoUrls": {
          "type": "array",
          "xml": {
            "name": "photoUrl",
            "wrapped": true
          },
          "items": {
            "type": "string"
          }
        },
        "tags": {
          "type": "array",
          "xml": {
            "name": "tag",
            "wrapped": true
          },
          "items": {
            "$ref": "#/definitions/Tag"
          }
        },
        "status": {
          "type": "string",
          "description": "pet status in the store",
          "enum": ["available", "pending", "sold"]
        }
      },
      "xml": {
        "name": "Pet"
      }
    },
    "ApiResponse": {
      "type": "object",
      "properties": {
        "code": {
          "type": "integer",
          "format": "int32"
        },
        "type": {
          "type": "string"
        },
        "message": {
          "type": "string"
        }
      }
    }
  },
  ...
}
```

![swagger-endpunkt.png](/images/2008/swagger-endpunkt.png "Generierter Abschnitt im PDF für die Swagger-Schnittstellenbeschreibung.")

Hierzu werden in der pom.xml Properties definiert, welche die Konfiguration rund um
das Plugin enthalten:

```xml
<properties>
    ...
    <!-- Swagger2Markup transformiert die swagger.json zu AsciiDoc-Dokumentation -->
    <swagger2markup.plugin.version>1.3.7</swagger2markup.plugin.version>
    <swagger2markup.extension.version>1.3.3</swagger2markup.extension.version>
    <swagger2markup.version>1.3.3</swagger2markup.version>
    <swagger2markup.generated-swagger-adoc-path>${project.build.directory}/swagger_adoc</swagger2markup.generated-swagger-adoc-path>
    <swagger2markup.swagger-json-path>${project.basedir}/src/main/resources/</swagger2markup.swagger-json-path>
    <swagger2markup.swagger-json-file>petstore-swagger.v2.json</swagger2markup.swagger-json-file>
</properties>
```

Und die Konfiguration des Plugins selbst wird im `pluginManagement`-Bereich der pom.xml definiert,
damit die Ausführung im Workflow je nach System anders aufgebaut werden kann.

```xml
<build>
    <pluginManagement>
        <plugins>
            ...
            <!-- Swagger2Markup transformiert die swagger.json zu AsciiDoc-Dokumentation -->
            <plugin>
                <groupId>io.github.swagger2markup</groupId>
                <artifactId>swagger2markup-maven-plugin</artifactId>
                <version>${swagger2markup.plugin.version}</version>
                <configuration>
                    <swaggerInput>${swagger2markup.swagger-json-path}/${swagger2markup.swagger-json-file}</swaggerInput>
                    <outputDir>${swagger2markup.generated-swagger-adoc-path}</outputDir>
                    <config>
                        <swagger2markup.markupLanguage>ASCIIDOC</swagger2markup.markupLanguage>
                        <swagger2markup.outputLanguage>DE</swagger2markup.outputLanguage>
                        <swagger2markup.pathsGroupedBy>TAGS</swagger2markup.pathsGroupedBy>
                        <swagger2markup.generatedExamplesEnabled>true</swagger2markup.generatedExamplesEnabled>
                    </config>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>io.github.swagger2markup</groupId>
                        <artifactId>swagger2markup-import-files-ext</artifactId>
                        <version>${swagger2markup.extension.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>io.github.swagger2markup</groupId>
                        <artifactId>swagger2markup</artifactId>
                        <version>${swagger2markup.version}</version>
                    </dependency>
                </dependencies>
                <executions>
                    <execution>
                        <phase>test</phase>
                        <goals>
                            <goal>convertSwagger2markup</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```


## Referenzen &amp; weiterführende Links

* [AsciiDoc](//asciidoc.org)
* [asciidoctor-maven-plugin](//github.com/asciidoctor/asciidoctor-maven-plugin)
* [PlantUML](//plantuml.com)
* [graphviz](//graphviz.org/download)
* [swagger2markup-maven-plugin](//github.com/Swagger2Markup/swagger2markup-maven-plugin)
* [petstore](//petstore.swagger.io)