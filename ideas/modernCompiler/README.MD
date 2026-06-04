# Modern Compiler

A design for a modern, extensible compiler with incremental parsing, enrichment, and lowering phases.

## Key Requirements

- Extendable grammar
- Incremental compilation
- Incremental parsing
- Error reporting
- Plugin support
- Sample implementation

---

## Architecture Overview

The compiler is split into three phases:

1. **Parsing** — source text → unenriched IR
2. **Enrichment** — lazy symbol resolution via a problem graph
3. **Lowering** — IR → target language output

This separation allows grammar extension and IR transformation at each stage.

---

## Phase 1: Parsing

Parsing produces an unenriched IR directly. Parsers are written as extension functions on a `SourceCharSequence`, making them composable and overridable.

```kotlin
fun MySourceCharSequence.Class(container: IrDeclarationContainer) = tryParse {
    CLASS() ?: return@tryParse null
    ws() ?: return@tryParse null
    val name = IDENTIFIER() ?: return@tryParse null
    val klass = IrClass(name, container)
    ws()
    if (OPEN_BRACE() != null) {
        ws()
        val declarations = Declarations(klass)
        declarations?.also { klass.declarations.addAll(it) }
        ws()
        CLOSE_BRACE()
    }
    return@tryParse klass
}

fun MySourceCharSequence.Declaration(container: IrDeclarationContainer) =
    Class(container) ?: Fun(container) ?: Var(container)

fun MySourceCharSequence.Declarations(container: IrDeclarationContainer): List<IrDeclaration>? =
    many({ Declaration(container) }, SourceCharSequence::ws)
```

### Extending the Grammar

To add a new construct (e.g. `alias`), override `Declaration` in your plugin and add the new parser:

```kotlin
fun MySourceCharSequence.Declaration(container: IrDeclarationContainer) =
    Class(container) ?: Fun(container) ?: Var(container) ?: Alias(container)

fun MySourceCharSequence.Alias(container: IrDeclarationContainer) = tryParse {
    val name = IDENTIFIER()
    // ... alias-specific parsing logic
}
```

Downstream resolution logic also needs updating to handle the new IR node.

---

## Phase 2: Enrichment

Symbols are resolved lazily — only when needed. This is modeled as a **problem graph**: each unresolved reference is a `ProblemNode` that knows how to resolve itself.

### Example: Java-like Source

```java
class A {
    void f() {}
}

class B extends A {
    void g() {
        f();
    }
}
```

### Corresponding IR

```kotlin
class JavaClass(
    var name: String,
    val superTypesDelegate: ProblemNode<JavaClass, List<JavaType>>,
    val declarationsDelegate: ProblemNode<JavaClass, List<JavaDeclaration>>,
) : JavaDeclaration, TypeSymbol {
    var parent: DeclarationContainer? = null
}

class JavaType(
    val symbol: ProblemNode<JavaType, TypeSymbol>,
) {
    var parent: Any? = null
}

class JavaMethod(
    val name: String,
    val returnType: ProblemNode<JavaMethod, TypeSymbol>,
    val body: ProblemNode<JavaMethod, JavaExpression>,
) : JavaDeclaration

class JavaBlock(
    val statements: List<ProblemNode<JavaBlock, JavaStatement>>,
) : JavaExpression

class JavaCall(
    val callee: ProblemNode<JavaCall, JavaCallable>,
) : JavaExpression
```

### ProblemNode Interface

```kotlin
interface ProblemNode<T, R> {
    val problemNode: NodeActionImpl<R>
        get() = NodeActionImpl { resolve() }

    fun T.resolve(): R
}
```

### Resolution Implementations

```kotlin
class SuperProblemNode(
    val initialTypes: List<JavaType>,
) : ProblemNode<JavaClass, List<JavaType>> {
    fun JavaClass.resolve(): List<JavaType> = initialTypes
}

class TypeProblemNode(
    val name: String,
) : ProblemNode<JavaType, TypeSymbol> {
    fun JavaType.resolve(): TypeSymbol =
        resolveAsGeneric()
            ?: resolveAsLocal()
            ?: resolveFromImports()
            ?: throw RuntimeException("Cannot resolve $name")
}

class CallableProblemNode(
    val name: String,
) : ProblemNode<JavaCall, JavaCallable> {
    fun JavaCall.resolve(): JavaCallable =
        resolveLocal()
            ?: resolveFromSuperTypes()
            ?: throw RuntimeException("Cannot resolve $name")
}

fun List<JavaDeclaration>.resolveClass(name: String): JavaClass? =
    filterIsInstance<JavaClass>().firstOrNull { it.name == name }
```

Methods like `resolveLocal` and `resolveFromSuperTypes` use context receivers to access the problem node:

```kotlin
context(c: ProblemNode<JavaCall>)
fun JavaCall.resolveLocal(): JavaCallable? {
    // walk scope chain and return matching callable, or null
}
```

### How the Problem Graph Works

- Resolving a method call may require resolving its enclosing class, which may require resolving superclasses, which may require resolving imports, and so on.
- Each `ProblemNode` encapsulates this dependency chain and resolves lazily.
- Plugins can access any resolved node; if something is missing, it can be resolved manually or added to the graph.
- Unresolved symbols surface as errors either during lowering or via a pre-lowering validation visitor.

---

## Phase 3: Lowering

Lowering walks the enriched IR and emits target-language output. Implement a visitor over the IR nodes:

```kotlin
// Example: lowering to bytecode or another IR
class AsmLoweringVisitor : IrVisitor {
    override fun visitClass(klass: JavaClass) { /* emit class structure */ }
    override fun visitMethod(method: JavaMethod) { /* emit method body */ }
    override fun visitCall(call: JavaCall) { /* emit call instruction */ }
}
```

Lowering is where all remaining unresolved problems must be resolved. Any `ProblemNode` that was not yet triggered will be forced here.

## Plugin API

Plugins expose two extension points:

- **Parser** — add new grammar constructs by overriding parser functions on `SourceCharSequence`.
- **Enrichment** — add new symbols and resolution strategies to the problem graph.
