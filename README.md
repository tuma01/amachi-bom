# Amachi BOM

Versiones y estándares de build de todo el ecosistema Amachi (Amachi Platform y las apps de cada sector). Sigue el
mismo diseño que Spring Boot (`spring-boot-dependencies` + `spring-boot-starter-parent`):

| Artefacto | Rol | Equivalente en Spring Boot |
|---|---|---|
| `com.amachi:amachi-dependencies` | BOM puro: solo versiones de dependencias | `spring-boot-dependencies` |
| `com.amachi:amachi-parent` | POM padre: hereda el BOM y agrega estándares de build y plugins | `spring-boot-starter-parent` |
| `com.amachi:amachi-build` | Agregador de este repositorio (nunca se usa como padre) | `spring-boot-build` |

| Versión | Java | Spring Boot | Spring Cloud | Spring AI | JUnit | Cucumber |
|---|---|---|---|---|---|---|
| `2.1.x` | 25 | 4.1.x | 2025.1.x | 2.0.x | 6.1 | 8.0 |
| `2.0.x` (`com.amachi.core:amachi-bom`, congelada para Vitalia) | 25 | 4.0.0 | — | — | 6.0 | 7.30 |

## Uso recomendado: heredar `amachi-parent`

```xml
<parent>
    <groupId>com.amachi</groupId>
    <artifactId>amachi-parent</artifactId>
    <version>2.1.0-SNAPSHOT</version>
</parent>
```

La app recibe:

- **Versiones** de Spring Boot, Spring Cloud, Spring AI, JUnit, Cucumber y de las librerías que Boot no gestiona
  (springdoc, jjwt, MapStruct, Instancio, WireMock, ArchUnit). Las dependencias se declaran **sin versión**.
  Una versión se puede ajustar en la app con su propiedad (ej. `<springdoc.version>`), solo con justificación.
- **Estándares**: Java 25, UTF-8 en código y `.properties`, nombres de parámetros en tiempo de ejecución, avisos de
  APIs obsoletas visibles, builds reproducibles, `build-info` para Actuator.
- **Plugins configurados**, que se activan declarándolos en la app sin versión: pruebas (Surefire, Failsafe, JaCoCo,
  PIT), estilo (Spotless), análisis (Checkstyle, PMD, SpotBugs + FindSecBugs, Sonar), cadena de suministro (OWASP
  Dependency-Check, SBOM CycloneDX), contenedores (Jib) e imagen nativa (GraalVM).

## Uso alternativo: importar solo las versiones

Para un proyecto que debe conservar otro padre:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.amachi</groupId>
            <artifactId>amachi-dependencies</artifactId>
            <version>2.1.0-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

En este modo las versiones no se pueden ajustar por propiedad y los estándares de build no se heredan.

## Reglas de mantenimiento

1. **Una línea por línea menor de Spring Boot** (`2.1.x` = Boot 4.1.x). Una versión publicada no se edita: las apps
   cambian la versión de su `<parent>` cuando están listas.
2. **Nunca fijar una librería que Spring Boot ya gestiona** (revisar `spring-boot-dependencies` antes): el pin local la
   degrada en silencio en la siguiente actualización de Boot.
3. Versiones de dependencias en `amachi-dependencies`; estándares, plugins y sus versiones en `amachi-parent`. Las
   versiones de plugins siguen el `pluginManagement` de `spring-boot-dependencies` de la versión de Boot.
4. Antes de subir de línea de Boot, eliminar los avisos de APIs obsoletas: cada versión menor de Spring Boot borra lo
   que la anterior marcó como obsoleto.
5. Las apps declaran dependencias en `<dependencies>` solo donde las usan; el padre nunca fuerza dependencias en todos
   los módulos.

## Construir

```bash
mvn clean install
```

Instala los tres POM en el repositorio Maven local (`~/.m2`). Instalar una versión nueva no reemplaza las anteriores.

## Variables de entorno

- `NVD_API_KEY`: clave de la API del NVD para OWASP Dependency-Check (solo para el análisis de vulnerabilidades).
