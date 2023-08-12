# Wasmi_example

## Functions of a runtime
1. Load wasm file
2. Validate format of the wasm file
3. Expose the exports of the wasm module
4. Make sure the Imports of the wasm module are satisfied
5. Compile the wasm module into a loadable executable
6. Execute the compiled wasm module
7. Enforce Isolation of the wasm module from other modules. 


## The Process
1. fetch the wasm binary file. You may have to covert it from wat or a stream of bytes. 
2. Validate the format of the wasm file.  
3. 
