# Erlang MMAP `emmap`

This Erlang library provides a wrapper that allows you to memory map files into the Erlang memory space.  

## Mix

add it to your mix dependencies

```elixir
defp deps do
  [
    {:emmap, github: "aler/emmap"}
  ]
end
```

## Basic Usage with Elixir

The basic usage is

```elixir
# iex -S mix

{:ok, mem} = :emmap.open('filename', [:read, :shared, :direct])
{:ok, binary} = :file.pread(mem, 100, 40)
...
:ok = :file.close(mem)
```

The open options is a list containing zero or more of these:

- `:read`, `:write`: Open for reading and/or writing (you can specify both).
- `:private`, `:shared`: The file is opened with copy-on-write semantics, or sharing memory with the underlying file.
- `:direct`: read/pread operations do not copy memory, but rather use "resource binaries" that can change content if the underlying data is changed.  This is the most performant, but also has other implications.
- `:lock`, `:nolock` do (or do not) use a semaphore to control state changes internally in the NIF library.  

From this point, `mem` can be used with the `file` operations

- `{:ok, binary} = :file.pread(mem, position, length)` read length bytes at position in the file.
- `:ok = :file.pwrite(mem, position, binary)` writes to the given position. 
- `{:ok, binary} = :file.read(mem, length)` read 1..length bytes from current position, or return `:eof` if pointer is at end of file.
- `{:ok, pos} = :file.position(mem, where)` see file:position/2 documentation.
- `:ok = :file.close(mem)`

## Notes

Using the option `:direct` has the effect that the mmap file is not closed until all references to binaries coming out of read/pread have been garbage collected.  This is a consequence of that such binaries are referring directly to the mmap'ed memory.  
