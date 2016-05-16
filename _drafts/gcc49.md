
I have been trying to build Grass GIS 7.0.4 on my Arch Linux box. I am using the latest packages often with AUR supplied packages as well. In this case I have the latest Grass GIS packages from aur, which are compiled from source.
There seems to be a [problem with Grass GIS 7.0.4 and GCC6](https://trac.osgeo.org/grass/ticket/2871) though. So I decided to install GCC 4.9 alongside GCC 6 and see if Grass GIS compiles with it.
This was a whole challenge in itself though. The AUR package for GCC 4.9 has it's own compilation problems. I more or less found a solution to this in a [post by Sidney Marchall at the gcc mailing-list](https://gcc.gnu.org/ml/gcc/2013-03/msg00299.html).
I created a PKGBUILD from this solution that should work for other people. To install GCC 4.9 clone [my PKGBUILD on github](https://github.com/dolfandringa/aur-gcc49/tree/master). Run `makepkg` in the folder and after compilation (which takes a while) run `pacman -U gcc49-4.9.3-1-x86_64.pkg.tar.xz`. This should install GCC 4.9 for you.

