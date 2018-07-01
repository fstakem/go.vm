
go.vm
-----

This project is a golang based compiler and intepreter for a simple virtual
machine.  It is a port of the existing project:

* https://github.com/skx/simple.vm

(The original project has a perl based compiler/decompiler and an interpreter
written in C.)

You can get a feel for what it looks like by refering to either the parent
project, or [the examples](examples/) contained in this repository.

This particular virtual machine is intentionally simple, but despite that it is hopefully implemented in a readable fashion.  ("Simplicity" here means that we support only a small number of instructions, and the 16-registers the virtual CPU possesses can store strings and integers, but not floating-point values.)


# Installation

Install the code via:

    $ go get -u  github.com/skx/go.vm
    $ go install github.com/skx/go.vm

Once installed there are three sub-commands of interest:

* `go.vm compile $file.in`
   * Compiles the given program into bytecode.
* `go.vm execute $file.raw`
   * Given the path to a file of bytecode, then interpret it.
* `go.vm run $file.in`
   * Compiles the specified program, then directly executes it.

So to compile the input-file `examples/hello.in` into bytecode:

     $ go.vm compile examples/hello.in

Then to execute the resulting bytecode:

     $ go.vm execute examples/hello.raw

Or you can handle both steps at once:

     $ go.vm run examples/hello.in


# Opcodes

The virtual machine has 16 registers, each of which can store an integer
or a string.  For example to set the first two registers you might write:

     store #0, "This is a string"
     store #1, 0xFFFF

In addition to this there are several mathematical operations which have
the general form:

     operations $result, $src1, $src2

For example to add the contents of register #1 and register #2, storing
the result in register 0 you would write:

     add #0, #1, #2

Strings and integers may be displayed to STDOUT via:

     print_str #1
     print_int #3

Control-flow is supported via `call`, `ret` (for subroutines) and `jmp`
for absolute jumps.  You can also use the `Z` register for comparisons and
make conditional jumps:

        store #1, 0x42
        cmp #1, 0x42
        jmpz ok

        store #1, "Something weird happened!\n"
        print_str #1
        exit
      :ok
        store #1, "Comparing register #01 to 0x42 succeeded!\n"
        print_str #1
        exit

Further instructions are available and can be viewed beneath [examples/](examples/).  The biggest omissions are probably reading from STDIN, printing a single (ascii) character.


# Notes

## compiler

The compiler is built in a traditional fashion:

* Input is split into tokens via [lexer.go](lexer/lexer.go)
  * This uses the [token.go](token/token.go) for the definition of constants.
* The stream of tokens is iterated over by [compiler.go](compiler/compiler.go)
  * This uses the constants in [opcode.go](opcode/opcode.go) for the bytecode generation

The approach to labels is the same as in the inspiring-project:  Every time
we come across a label we output a pair of temporary bytes in our bytecode.
Later, once we've read the whole program and assume we've found all existing
labels,  we go back up and fix the generated addresses.

You can use the `dump` command to see the structure the lexer generates:

     $ go.vm dump ./examples/hello.in
     {STORE store}
     {IDENT #1}
     {, ,}
     {STRING Hello, World!
     }
     {PRINT_STR print_str}
     {IDENT #1}
     {EXIT exit}


## interpreter

The intepreter is located in the file [cpu.go](cpu/cpu.go) and is
as simple and naive as you would expect.

Steve
--
