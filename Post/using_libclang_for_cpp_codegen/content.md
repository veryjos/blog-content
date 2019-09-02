## The Goal

Bind Javascript ([ChakraCore](https://github.com/microsoft/ChakraCore)) to C++ with minimal boilerplate, and consistent conventions.

Instead of using template metaprogramming, I set up a compile-time build step written in Python using `libclang`. This build step parses your source, seeks out specific annotations, and outputs wrapper classes which bind your class to ChakraCore so it can be used in JavaScript.

This is the same approach that **Qt's moc and Unreal Engine 4** take-- a separate, custom build step to generate reflection information before compilation.

## The Tradeoff

Don't get me wrong-- I get it. Preprocessor steps, transpilation, and the like are warning signs that your development ecosystem is contracting a terminal illness. I'll be the first to say I've ignored projects outright if they do things like this.

But this time, it makes sense. I swear. The value we're getting back is huge.
  - Standard calling and binding conventions
  - Shorter compile times
  - Abstract (Could just as easily generate bindings for C#, Python, etc. just by replacing the template)
  - Easier to implement and maintain

The downsides are decreased portability for the build environment, and non-standard syntax for users.

## Example

```c++
/**
 * Bind a simple "Entity" class
 * CC::Bind    = Bind to JS
 * CC::Virtual = Allow this class to be extended with an ES6 class
 **/
Meta(CC::Bind, CC::Virtual)
struct Entity {
  // Bind the default ctor (can be overloaded)
  Meta(CC::Bind)
  Entity()

  // Bind some attack function as virtual
  Meta(CC::Bind, CC::Virtual)
  virtual void Attack();

  // Bind the UID for this entity as read-only
  Meta(CC::Bind, CC::ReadOnly)
  uint32_t id;

  // Bind the entity's position
  Meta(CC::Bind)
  Vec2 position;
}
```

This is how **easy** it can be to bind code from CPP <-> JS. Typings are generated for **TypeScript** users as well.

The macros used for annotating are ignored when your compiler generates the binary. They're only understood by the preprocessor I wrote with `libclang`.

In your JavaScript or TypeScript, you can now use the bindings like this:

```js
() => {
  class Player extends Entity {
    constructor() {
      super();
    }

    // Override the virtual attack method with duck typing
    Attack() {
      console.log(`I'm attacking at ${this.x} ${this.y}`);
    }
  };

  return new Player();
}
```

If you were to call this JavaScript function from C++, you would get a normal `Entity` pointer back, and it can be used normally. By default, the calling code owns the memory and must free it.

Easy, right? You can even define your own primitive types using shims written in Python.

## Performance

Performance when talking between two languages is never so great. Crossing ABI boundaries implies marshalling, and makes inlining optimizations or LTO impossible.

To help remedy that, I expose special binding type called mirror bindings.

## Mirror Bindings

`CC::Mirror` should be used on something like a Vec2-- it's **lazy marshalling**. The Vec2 will live in C++ or JS exclusively, with no vtable bloat or inheritance required, until it has to cross the ABI boundary.

Then, it will be marshalled into the host or hosted language's representation-- where it will again be treated as a pure native type until it needs to cross back across boundaries.

This is good because it allows ChakraCore to inline code for this object, and allows C++ to do inlining and LTO at compile-time.

The drawback is type slicing- when crossing between ABI's, the type must be sliced down into its most basic class, so extending these objects is **impossible**. This is much like type slicing in C++ when copying objects.

You also incur the marshalling penalty every time you pass the object between ABI's, as opposed to at access time.

## Conclusion

Initial thoughts? It's pretty good for my project.

As time moves on, we'll see how well this approach scales. As we speak, I'm writing a DSL for creating bindings, just in case portability or preprocessing time really takes a hit. After all, it's C++. The entire translation unit must be parsed before preprocessing can even begin.

Only time will tell!
