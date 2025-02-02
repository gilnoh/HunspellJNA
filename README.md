[![Project Status: Active - The project has reached a stable, usable state and is being actively developed.](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active)


# What is this fork?
- HunspellJNA is nice tool, but main fork is outdated and do not support ARM64 (aarch64): e.g. no native binaries for M1 mac, or AWS Graviton instances. 
- This fork includes those pre-built aarch64 binaries in /native-lib, and updated JNA dependencies. So you can build maven artifact Jar that runs okay in aarch64 systems.  
- More specifically, the followings have been changed 
  - added pre-built aarch64 binary for Darwin (From Hunspell 1.7 - from Homebrew pre-built cask) 
  - added pre-built aarch64 binary for Linux (From Hunspell 1.6.2 - from Debian ARM64 package)
  - HunspellJNA java code updated to recognize and load correct aarch64 libraries
  - Updated some required dependencies (e.g. JNA to latest version, so aarch64 loading works okay)  
- building a maven package from this branch will produce a fully working JAR including the following systems 
  - M1 macs, native aarch64 java 
  - AWS graviton instances (e.g. m6g instances or equivalents)
  - (and all other platforms worked before) 
- WARNING
  - The changes that we made here is really minimal, and we do not expect this to be merged on upstream as-is. 
  - For example, native lib build script has NOT been updated; only pre-built binaries have been added for your / our convenience. 


# (ORIGINAL DOCUMENTATION) --- JNA based Java API for Hunspell

This package contains the JNA based Java bindings for the Hunspell, see:
https://github.com/twall/jna and http://hunspell.sourceforge.net/ for details
on how these work.

This java wrapper lives at:
https://github.com/dren-dk/HunspellJNA

See this page for dictionaries:
http://wiki.services.openoffice.org/wiki/Dictionaries

# Included native binaries 

I'm in the process of obtaining the binaries for new versions of hunspell,
at the moment these are the versions included:

## 1.6.2
* libhunspell-linux-x86-64.so

## 1.3.2
* hunspell-darwin-ppc-32.jnilib
* hunspell-win-x86-32.dll
* hunspell-win-x86-64.dll
* libhunspell-darwin-x86-32.dylib
* libhunspell-darwin-x86-64.dylib
* libhunspell-linux-x86-32.so



# Building the native binaries

Before using the java API you must build some native binaries to include
in the jar, to make things easy I've included the binaries from the platforms
that I care about in native-lib.

The binaries in native-lib were built on the various platforms using the
native-build.pl script, the source for the native libs is in native-src and
consists of an unmodified hunspell source tar ball and any needed diffs.

I've included a script called native-build.pl which might help to build
on at least Linux and 32 bit OSX/windows, for 64 bit OSX/windows you
can take it as a hint of what to do.

Compiling libhunspell for 64 bit OSX is a pain, see readme-darwin-64bit.txt
for the changes that are needed to make it work.

Compiling libhunspell for 64 bit windows means having to change the vc project
to compile for the x64 platform.


# Building the Java API

To build `hunspell.jar` simply run `ant`. This will produce
`build/jar/hunspell.jar` which contains everything a standalone application
could want (aside from `jna.jar` which can be found on the JNA page or in lib)


# Building the Java API for use via webstart

If you plan on using `hunspell.jar` from a webstart application run:
`ant webstart`.

This will produce two native jar files per supported platform (one for JNA
and one for hunspell) as well as pure java `jna-jws.jar` and `hunspell-jws.jar`
in `build/jar`.

Include `jna-jws.jar` and `hunspell-jws.jar` in the common resources section in
normal jar tags, the platform specific binaries must go into platform specific
resource sections as nativelib entries.


# Working around a bug in nativelib handling of webstart

I had great trouble getting nativelib with more than one jar file to work on
osx and linux (it worked fine on windows), there are two workarounds:

1) Use plain `<jar/>` tags in stead of `<nativelib/>`. This will cause both JNA and
   hunspell to search the classpath for the needed binaries and they will then
   extract the binaries and load them as usual.
   This approach might leak a binary each run on windows because of
   the mandatory file locking used on that platform.

2) Consolidate all your nativelib files for each platform into one.
   This looks slightly less pretty, but it works and is slightly faster during
   download as well.


# Output

The output of `ant` is one large jar file containing binaries for all the supported platforms, it's about 800k, so it's not very nice for folks who have to download it though: 

| Name				| Description					|
|-------------------------------|-----------------------------------------------|
| hunspell.jar			| the Java API + all binaries			|
| hunspell-jws.jar		| just the Java API, for use in webstart	|
| hunspell-darwin-i386.jar	| binaries for use as OS specific resources	|
| hunspell-darwin-ppc.jar	|						|
| hunspell-linux-amd64.jar	|						|
| hunspell-linux-i386.jar	|						|
| hunspell-win32-x86.jar	|						|

I've parted out the lib/jna.jar file in the same manner, also for use in webstart: 

| Name			| Description		|
|-----------------------|-----------------------|
jna-jws.jar             | the Java API for JNA	|
jna-darwin.jar          |			|
jna-linux-amd64.jar	|			|
jna-linux-i386.jar	|			|
jna-win32-x86.jar	|			|


# API

See `HunspellMain.java` for a very simple example of how to use the API, it
boils down to:

```
Hunspell.Dictionary dict = 
	Hunspell.getInstance().getDictionary("/path/to/your/unzipped/dictionary/en_US"); 

if (dict.misspelled("wrod")) {
	ArrayList<String> suggestions = dict.suggest("wrod");
	 ...
}
```

IOW: The application must also supply the two dictionary files (.dic and .aff) found in the zip file unzipped on disk and point at them using the full path, plus  the part of the file name before .dic. 

`Hunspell.getInstance()` caches the loaded hunspell library, so there is no overhead in calling it more than once.

`getDictionary()` is also internally cached, so it costs no more than a hash lookup when calling the second time.

# Testing

For a quick test of the JNA interface run

```
java -cp lib/jna.jar:build/jar/hunspell.jar\ 
-Droot=/path/to/your/dicts/ -Dlang=en_GB dk.dren.hunspell.HunspellMain
```


# TODO

`getDictionary()` doesn't re-load the files once they are loaded, it should probably do that automagically if it notices that the files have changed on disk.


# Thanks

Although this code doesn't contain any, it's very much inspired by the JNA bindings in OmegaT, so thank you to the OmegaT developers.

Andrzej Zydron figured out how to build libhunspell for 64 bit OSX.

# Misc

At the moment the package contains copies of:
* jna.jar version 3.4.0 
* Hunspell version 1.3.2 


Patches and suggestions are welcome.

Flemming Frandsen (flfr@stibo.com / ff@nrvissing.net) 
