# Changes from original
- Removed all pre-compiled dependencies, other than FMOD.
Using VCPKG for the dependencies allows us to link to the debug versions of the libraries when performing debug builds versus always using the release libraries. 
- Added https://github.com/sonoro1234/luafilesystem because luajit was complaining about lfs missing.
- the Bullet library profile now is disabled if the bullet library hasn't been compiled to include (VCPKG has it disabled by default).

# TODO
- Bullet3 can be compiled with a flag to use doubles instead of floats everywhere. PHX does not support that yet and requires the 32bit float version.
- Figure out how to use conan as an alternative for anyone that doesn't like vcpkg.

# Original README below
# LibPHX

LibPHX is a lightweight, (mostly) C-based game engine developed for the cancelled space simulation game Limit Theory. LibPHX focuses on implementing the core features necessary to build performant 3D games without including the kitchen sink.

Rather than acting as a framework like most modern game engines, the LibPHX philosophy is to provide a game engine *as a library* -- that is, control flow is in the hands of the user, not the engine. LibPHX is designed specifically to be controlled from Lua scripts using the LuaJIT runtime. The engine's C interface, combined with LuaJIT's FFI technology, allows for scripts to make zero-overhead calls into the engine, thus allowing *the majority* of game logic and control to remain in script. Keeping control flow in the hands of Lua opens up a world of fast iteration and fully-moddable game logic, while achieving maximal performance by delegating heavy computation to engine-side functions.

LibPHX is now abandoned and mostly undocumented, however you can see examples of its usage in https://github.com/JoshParnell/ltheory.
