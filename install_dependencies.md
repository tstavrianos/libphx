**Using VCPKG:**

`vcpkg install freetype SDL2 bullet3 glew lz4 luajit --triplet=x64-windows`

Replace x64-windows with the triplet of your choice.
- I've tested and confirmed that x64-windows works (on my machine at least).
- Haven't been able to get x64-windows-static to work yet (bullet doesn't want to link properly).
- Haven't tested osx or linux triplets because I haven't got a suitable machine to test them on.

**Using Conan:**

`TODO`