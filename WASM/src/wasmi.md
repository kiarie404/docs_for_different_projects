# Wasmi

# Hosting off the browser

## Let's first discuss the wasmi-core crate
The wasm_virtual machine is a 32-bit architecture. Each page has 2^16 bytes.  
Meaning that the RAM(linear memory) can have a maximum of 2^16 pages. With Each page being 64Kb(2^16 byte)  

Under some conditions, Wasm execution may produce a Trap, which immediately aborts execution. Traps cannot be handled by WebAssembly code, but are reported to the host embedder.  

TrapCode : Error type which can be thrown by wasm code or by host environment.  
Here are all the traps that can be thrown during excecution :  
```rust
pub enum TrapCode {
    UnreachableCodeReached, // Wasm code executed unreachable opcode.
    MemoryOutOfBounds, // Attempt to load or store at the address which lies outside of bounds of the memory.
    TableOutOfBounds, // Attempt to access table element at index which lies outside of bounds.
                        // This typically can happen when call_indirect is executed with index that lies out of bounds.
    IndirectCallToNull, // Indicates that a call_indirect instruction called a function at an uninitialized (i.e. null) table index.
    IntegerDivisionByZero,
    IntegerOverflow, // Attempted to make an invalid conversion to an integer type
    BadConversionToInteger,
    StackOverflow, // This is likely caused by some infinite or very deep recursion. Extensive inlining might also be the cause of stack overflow.
    BadSignature,  // Attempt to invoke a function with mismatching signature.
    OutOfFuel,  // This is specific to wasmi in ethereum
}
```

This errors are advanced, pay no attention to them. Just know that the wasm code section might generate exceptions at runtime and that those exceptions are wrapped as Traps that can be handled by the Host code section ONLY.  

## Time to discuss the wasmi crate
wasmi abstracts the entire virtual machine. You can use this abstraction as a crate that can be embedded inside host code.  



# Random
- can only parse wasm files
- cannot parse wat files but we are into wasm files.... so this is not an issue

- can validate a .wasm file
- The engine is the one that does the execution.  

- A store is like the schema used to store Host data that the wasm module can access.  

#### An Engine
An Engine is the wasmi intepreter. I imagine that it is a program that supplies the OS with the final machine instructions in an organized manner.  
There can be many engines being ran by the Host code. So always check if references of two engines are the same using the function below
```rust
pub fn same(a: &Engine, b: &Engine) -> bool
```
You can :
1. create a new engine with either default settings OR with custom configuration. Configuration information gets stored in a struct called Config.
2. You can extract the Config reference of an engine
3. You can compare if two engine references are actually referencing the same Engine

#### Config
The default configs are all good. We'll get into the advanced configs later.  

#### Export
This struct is An exported WebAssembly value.  
As earlier stated, a wasm module can only export : A table, A global variable, A function or a Linear Memory. All these values are called Extern enums.    
An export has the same lifetime as the wasm instance that defined that export.  
An export struct encloses the values (memory, Table, Global, Func). You can extract the values from the Export Struct  

You can get an iterator of Exports from an Instance using the Instance::exports function.  
You can get an iterator of Exports from a Module using the Module::exports function.

#### Store
A store is a struture that holds all data related to a WASM module. This is the userData. This is NOT the Linear Memory  
So the store can store data from the host that you need the wasm module to access.  
Structure the store like a schema. Wasmi calls it UserState

I do not know if you can have multiple Stores for one Module that has been associated to one Engine.  
You can :  
1. Create a new store associated with a certain Engine reference and define its type_schema...(Remember a store does not get associated to a single module, instead, it gets associated to an engine that might be running many modules. So this many modules can commuicate through a single store) 
2. Access the data inside the store : either as a reference or mutable reference.  
3. You can get the reference of the Engine that has been associated with a certain Store.

#### StoreContextMut and StoreContext
StoreContextMut is a temporary handle to a &mut Store<T>.  
StoreContext is a temporary handle to a & Store<T> 

#### Trait : AsContextMut
This trait lets you define which StoreContextMut your object will have access to.  
For example, if you define this trait for a Function Object, then that function will have temporarily have exclusie rights to the referenced Store<T>

This is what wasmi keeps referring to as context.  

### FuncType
This is a functions signature. It defines an iterator of parameters and an iterator of return types
You can :
1. define a new signature
2. get the params of an existing signature
3. get the results of an existing signature 

#### Func
This is a reference to a function. It can be a host function or a wasm fuction.  

1. You can create a new function reference... and in this case, we will create a reference to a host function.  
2. You can Create a new host function from the given closure.  


##### 1. Creates a new Func with the given arguments.
```rust
pub fn new<T>(
    ctx: impl AsContextMut<UserState = T>, // which Store do you want to temporarily have exclusive rights to?
    ty: FuncType,
    func: impl Fn(Caller<'_, T>, &[Value], &mut [Value]) -> Result<(), Trap> + Send + Sync + 'static
) -> Self
```
Creates a new Func with the given arguments.  


#### Caller's Context
A caller's context is the metadata about the scope of the caller. For example If there is a wasm module A calling a host function. THen information about Module A can be wrapped up as a context. Metadata includes :
1. All exported items by name. You can fetch them by name
2. A reference to the engine that the caller is running on top of
3. A reference to the user-provided host data --> I don't get this


#### Exported Wasm Items
A wasm module can export : A Global variable, Linear memory, A function, A Table

#### Module
This is A parsed and validated WebAssembly module.  
A module gets associated with an Engine.  
You can extract the exports and imports as iterators from a module.     

#### Instance
An execute-ready wasm Module. All that is remaining is to let the Engine interpret it.  
Instances are owned by a Store. [I don't get this] 
Create new instances using Linker::instantiate. 

#### InstancePre
An Instance that has not yet started running. It is up to you to press the 'start button'... OR Panic because maybe the Instance was not supposed to have a 'start' function

#### Linker
A linker used to define module imports and instantiate module instances.    
A linker gets associated with an Engine. It is not independent.  

You can :
1. Define item of the symbol table

## Wat Rust library
A Rust parser for the WebAssembly Text format.  

This crate contains a stable interface to the parser for the WAT format of WebAssembly text files. The format parsed by this crate follows the online specification.  

Its primary function is to parse a wat data and produce the corresponding wasm binary.  
It can : 
1. parse a string of bytes (wat as bytes) and produce a wasm binary
2. parse a wat file
3. parse a wat str
4. If it is supplied with a wasm file/string/bytes, it returns the same wasm file/string/bytes

It will throw Errors if any are found. 
1. The wat input may fail to lex, such as having invalid tokens or syntax
2. The wat input may fail to parse, such as having incorrect syntactical structure
3. The wat input may contain names that could not be resolved
4. if the input does not start with b"\0asm" and is invalid utf-8. 
