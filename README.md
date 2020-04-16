# OrganizeImports

A Scalafix custom rule that organizes import statements.

## Installation

To try this rule in your SBT console (with Scalafix enabled):

```
sbt> scalafix dependency:OrganizeImports@com.github.liancheng:organize-imports:<VERSION>
```

To use this rule in your SBT build:

```
ThisBuild / scalafixDependencies += "com.github.liancheng" %% "organize-imports" % "<VERSION>"
```

## Features

### Grouping global imports

This rule only organizes global imports appearing at the top of the source file. It handles both fully-qualified imports and relative imports but in different manners, because fully-qualified imports are order insensitive while relative imports are order sensitive. For example, sorting the following imports in alphabetical order would introduce a compilation error:

```scala
import scala.util
import util.control
import control.NonFatal
```

#### Fully-qualified imports

Import groups for fully-qualified imports are configured via the `groups` option. Each import group is defined by an import expression prefix pattern, which can be one of the following:

1.  A plain-text pattern

    E.g.: `"scala."`, which matches imports referring the `scala` package. Please note that the trailing dot is necessary, otherwise you may have `scalafix` and `scala` imports in the same group, which is not what you want in most cases.

1.  A regular expression pattern starting with `re:`

    E.g.: `"re:javax?\\."`, which matches both `java` and `javax` packages.

1.  `"*"`, the wildcard group.

    The wildcard group matches everything not belonging to any other groups. It can be omitted when it's the last group. So the following two configurations are equivalent:

    ```hocon
    OrganizeImports.groups = ["re:javax?\\.", "scala.", "*"]
    OrganizeImports.groups = ["re:javax?\\.", "scala."]
    ```

Example:

- Configuration:

  ```hocon
  OrganizeImports.groups = ["re:javax?\\.", "*", "scala."]
  ```

- Before:

  ```scala
  import java.time.Clock
  import scala.collection.JavaConverters._
  import sun.misc.BASE64Encoder
  import scala.concurrent.ExecutionContext
  import javax.annotation.Generated
  ```

- After:

  ```scala
  import java.time.Clock
  import javax.annotation.Generated

  import sun.misc.BASE64Encoder

  import scala.collection.JavaConverters._
  import scala.concurrent.ExecutionContext
  ```

#### Relative imports

Due to the fact that relative imports are order sensitive, they are moved into a separate group located after all the other groups, _with the original import order reserved_.

Example:

- Configuration:

  ```hocon
  OrganizeImports.groups = ["scala.", "*"]
  ```

- Before:

  ```scala
  import scala.collection.JavaConverters._
  import sun.misc.BASE64Encoder
  import scala.concurrent.ExecutionContext
  import scala.util
  import util.control
  import control.NonFatal
  ```

- After:

  ```scala
  import scala.collection.JavaConverters._
  import scala.concurrent.ExecutionContext
  import scala.util

  import sun.misc.BASE64Encoder

  import util.control
  import control.NonFatal
  ```

### Exploding import statements with multiple import expression

Example:

- Before:

  ```scala
  import java.time.Clock, javax.annotation.Generated
  ```

- After:

  ```scala
  import java.time.Clock
  import javax.annotation.Generated
  ```

### Exploding grouped import selectors into separate import statements

Example:

- Configuration:

  ```hocon
  OrganizeImports.importSelectorsPolicy = Explode
  ```

- Input:

  ```scala
  import scala.collection.mutable.{ArrayBuffer, Buffer, StringBuilder}
  ```

- Output:

  ```scala
  import scala.collection.mutable.ArrayBuffer
  import scala.collection.mutable.Buffer
  import scala.collection.mutable.StringBuilder
  ```

### Grouping import statements with common prefix

Example:

- Configuration:

  ```hocon
  OrganizeImports.importSelectorsPolicy = Group
  ```

- Before:

  ```scala
  import scala.collection.mutable.Buffer
  import scala.collection.mutable.StringBuilder
  import scala.collection.mutable.ArrayBuffer
  ```

- After:

  ```scala
  import scala.collection.mutable.{ArrayBuffer, Buffer, StringBuilder}
  ```

### Sorting import selectors

Import selectors within a single import expression can be sorted by a configurable order provided by the `importSelectorsOrder` enum option:

1.  `Ascii`

    Sort import selectors by ASCII codes, equivalent to the [`AsciiSortImports`](https://scalameta.org/scalafmt/docs/configuration.html#asciisortimports) rewriting rule in Scalafmt.

    Example:

    - Configuration

      ```hocon
      OrganizeImports {
        importSelectorsOrder = Ascii
        importSelectorsPolicy = Keep
      }
      ```

    - Input:

      ```scala
      import foo.{~>, `symbol`, bar, Random}
      ```

    - Output:

      ```scala
      import foo.{Random, `symbol`, bar, ~>}
      ```

1.  `SymbolsFirst`

    Sort import selectors by the groups: symbols, lower-case, upper-case, equivalent to the [`SortImports`](https://scalameta.org/scalafmt/docs/configuration.html#sortimports) rewriting rule in Scalafmt.

    Example:

    - Configuration

      ```hocon
      OrganizeImports {
        importSelectorsOrder = SymbolsFirst
        importSelectorsPolicy = Keep
      }
      ```

    - Input:

      ```scala
      import foo.{Random, `symbol`, bar, ~>}
      ```

    - Output:

      ```scala
      import foo.{~>, `symbol`, bar, Random}
      ```
1.  `Keep`

    Do not sort import selectors.

## Default configuration

```hocon
OrganizeImports {
  groups = ["re:javax?\\.", "scala.", "*"]
  importSelectorsOrder = Ascii
  importSelectorsPolicy = Explode
}
```
