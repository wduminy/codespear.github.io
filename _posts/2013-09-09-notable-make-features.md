---
layout: post
category: programming
tags: [C++,make]
---
The need for something to help with the compile process has been long recognised.  Few can disagree that the good old [_make_](http://www.gnu.org/software/make/manual/make.html) command line utility was a ground breaker in its day.  


The availability of alternatives leaves one with the impression that this program does not fit the bill for today's software. This assumption is far from the truth.

If you disagree you may not have given _make_ enough of your time to prove itself. Clearly there are situations where the features of another builder serves much better; but sometimes _make_ would do the job just fine.  It is after all very stable and available *by default* on many platforms.  

I do not want to create an introductory post. For that, please try the [good articles by Alex Allain](http://www.cprogramming.com/tutorial/makefiles_continued.html). For discerning minds there is the complete [_make_ manual](http://www.gnu.org/software/make/manual/make.html) - really an excellent reference.

What I want to show you is a real world example of how _make_ can be used to build a C++ program. The listing below is an early version of the [GameEx](https://github.com/codespear/GameEx) *makefile*.  This project has sources in multiple directories and consists of [static libraries](http://en.wikipedia.org/wiki/Static_library) and executable programs that use those libraries.   Although I use  [MinGW](http://www.mingw.org/),  the listed *makefile* might also work in *linux*.  But the idea of this post is not to write a portable *makefile*, it is simply to demonstrate what can be done with _make_.

{%highlight makefile linenos%}
TARGET_DIR = RunDir
COMPILE_ARGS = -I"SDL-1.2.15\include" -ITutLib -IGameLandLib -O3 -Wall -c -fmessage-length=0 -std=c++0x
LINK_ARGS = -L"SDL-1.2.15\lib" -L$(TARGET_DIR) -lmingw32 -lwsock32 -lglu32 -lopengl32 -lSDLmain -lSDL_ttf -lSDL.dll
DIRS = GameLandLib GameLandTests TutLib TerrainDemo Sneaky
SOURCES := $(foreach e, $(DIRS), $(wildcard $(e)/*.cpp))
DEPS := $(patsubst %.cpp, %.depends, $(SOURCES))
OBJS := $(patsubst %.cpp, %.o, $(SOURCES))
GAME_LAND_LIB := $(TARGET_DIR)/libGameLandLib.a
TUT_LIB := $(TARGET_DIR)/libTutLib.a
GAME_LAND_TEST := $(TARGET_DIR)/GameLandTests.exe
TERRAIN_DEMO := $(TARGET_DIR)/TerrainDemo.exe
SNEAKY := $(TARGET_DIR)/Sneaky.exe
.PHONY : clean all run_sneaky run_terrain run_test

run_sneaky: $(SNEAKY)
	cd $(TARGET_DIR); ./Sneaky.exe

$(SNEAKY): $(OBJS) $(GAME_LAND_LIB)
	g++ $(wildcard Sneaky/*.o) -o $@ -lGameLandLib $(LINK_ARGS)  

run_terrain: $(TERRAIN_DEMO)
	cd $(TARGET_DIR); ./TerrainDemo.exe

$(TERRAIN_DEMO): $(OBJS) $(GAME_LAND_LIB)
	g++ $(wildcard TerrainDemo/*.o) -o $@ -lGameLandLib $(LINK_ARGS)  

run_tests:  $(GAME_LAND_TEST)
	cd $(TARGET_DIR); ./GameLandTests.exe

$(GAME_LAND_TEST): $(OBJS) $(TUT_LIB)
	g++ $(wildcard GameLandTests/*.o) -o $@ -lGameLandLib -lTutLib $(LINK_ARGS)  

$(GAME_LAND_LIB): $(OBJS)
	ar -r $@ $(wildcard GameLandLib/*.o)

$(TUT_LIB): $(GAME_LAND_LIB) 	
	ar -r $@ $(wildcard TutLib/*.o)

clean:
	rm -f $(foreach e, $(DIRS), $(wildcard $(e)/*.depends-)) $(foreach e, $(DIRS), $(wildcard $(e)/*.o))

%.o: %.cpp
	g++ $(COMPILE_ARGS) -c $< -o $@

%.depends: %.cpp
	g++ -MT $(@:.depends=.o) -MM $(COMPILE_ARGS) $< -o $@

-include $(DEPS)
{%endhighlight%}

From the listing you can see the use of *macros* and *target definitions*.  On line 5 I make use of [the `foreach` function](http://www.gnu.org/software/make/manual/make.html#Foreach-Function) to make a list of all the `cpp` files in the given `DIRS`.  Note that there is no need to explicitly list the files that must be compiled. The macros in line 6 and 7 uses `patsubst` function used in line 6 and 7 to create useful target lists from the list of `cpp` files.   

You will also notice that I did not list the dependencies of the source files.  For this `%.depends` target definition (on line 45) uses the g++ `-MT` and `-MM` flags to create a depends file. See the [GCC documentation](http://gcc.gnu.org/onlinedocs/gcc-4.3.6/gcc/Preprocessor-Options.html#Preprocessor-Options) for more information on this flag. The `$(@:.depends=.o)` may look a bit cryptic at first glance.  

The expression `$(@:.depends=.o)` in line 46 is quite a nice _make_ trick. The `@` is a _special macro_ that contains the target name (e.g. `mydir\foo.depends`), the expression `$(a,b=c)` takes the value `a`, finds all occurrences of `b` in it and replace those with `c`. So `mydir\foo.depends` becomes `mydir\foo.o`.  

Note that the last line in the *makefile* has the effect to include all the generated `.depends` files into the _make_ definition.  

There is a subtle problem with this solution for managing dependencies, and that is that the dependencies are not regenerated.  So when you add a new header, you'll have to call `make clean` to delete all the old `.depends` files.  The next call to _make_ will create new ones, and your dependencies will be correct again.  There could be a solution for this, but I am satisfied with things as they are.

This file could make use of more macros, but it is sometimes wise to keep things readable - even if there is a bit of duplication. Apparently [recursive makefiles are harmful](http://miller.emu.id.au/pmiller/books/rmch/): this *makefile* show how simple it is to build sources from more than one directory.

That wraps it up.  Hope you now know a bit more about *making* source code.         
