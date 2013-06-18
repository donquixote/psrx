psrx
====

Alternative proposal for PSR-X autoloader spec (under construction)

Details can be discussed.

Compared to the [existing proposal](https://github.com/php-fig/fig-standards/blob/3740bbe873c8e31d0d218b5d1cd8fbb3cb82806a/proposed/autoloader.md),
this one attempts to
- set an explicit interoperability goal
- identify the audience and relevant parties for this interoperability goal
- provide an explicit distribution of responsibilities for those parties
- provide a guaranteed profit, if all parties do their job correctly
- provide loophole-free exact definitions
- be permissive, without being ambiguous or self-contradicting
- be explicit about its scope limitation
- provide definitions that can be translated into (unit) tests, if we choose to do so.


Open to discuss:
- We can try to make it less "mathy". But I would like to keep it equivalent.
- Whether a file can do other things than define the class expected by the class loader. This might be ok "at your own risk".
- The term "mapping" is a bit odd to describe only one pair of ($D, $N).
- Maybe we should have a term for a collection of ($D, $N) pairs? These could have an additional restriction to avoid intersections.
- In "Instructions for library developers", paths are relative to the library root.
  In other sections, these paths are absolute, or relative to the application root.
  The transition from one to the other needs to be clarified.


## The Spec

### Terminology

"PHP library" can refer to any collection of PHP files that is shipped at-once, meant to be included from other PHP code.  

"Application" means a collection of PHP files, of which some are meant to be executed directly, and others are meant to be included from there.
An application may contain a number of 3rd party libraries, and some code of its own.

We may also regard the application a "library" of its own, if it contains PHP files to be included from other PHP code.

We assume that libraries contain classes that, in an application context, should be autoloaded using PHP's spl autoload mechanism.

We also assume that the application developer wants to use one shared autoloader for all or most of the libraries.

"Autoloader" can refer to a system that can provide autoload callbacks for the spl autoload stack, or it can refer to such a callback directly.
An autoloader can be made up of PHP files, which by themselves will be autoloaded. How exactly this happens, is not subject for this document.


### Goal

The spec deals with three questions:
- How to write a PHP library, so that it can be made to work with any PSR-X compliant class loader.
- How to write a class loader, so that it can be made to work with any PSR-X compliant PHP library.
- How to write an application to use a shared PSR-X autoloader for a number of PSR-X libraries.


### Definition: PSR-X match

Let
- $D be a filesystem directory.
- $N be a valid PHP namespace.
- $F be a valid file path.
- $C be a valid fully-qualified PHP class name.

Then ($D, $N, $F, $C) is a PSR-X match, IF there are two strings $path_suffix and $class_suffix, such that:
- $class_suffix is the string that results from replacing every directory separator in $dir_suffix with a namespace separator.
- $dir_suffix is the string that results from replacing every namespace separator in $class_suffix with a directory separator.
- $F = $D + '/' + $path_suffix + '.php'.
- $F is an existing file.
- $C = $N + '\' + $class_suffix.


### Definition: PSR-X path-namespace mapping

Let $D be a filesystem directory, $N be a PHP namespace.  
($D, $N) is a PSR-X path-namespace mapping, IF  
For every file $F,  
if there is a fully-qualified class name $C,  
such that ($D, $N, $F, $C) is a PSR-X match,  
then the inclusion of $F from a PHP script
- must make $C available as a class or interface.
- (must not have any other side effects (under discussion))
- must NOT cause the script to crash, exit, or raise any errors or exceptions, UNLESS
  - A class or interface named $C was already defined at the time the file is included.
  - A base class or interface is not available.


### Instructions for library developers

Library developers who want to make their classes available for PSR-X autoloading should provide
- a directory structure with PHP files (that define those classes)
- a number of mappings ($D[1], $N[1]), .., ($D[n], $N[n])

such that for every ($D, $N):
- $D is a file path relative to the library root, specifying an existing directory in the library.
- $N is a valid PHP namespace name (of any level).
- ($D, $N) is a PSR-X namespace mapping.

The path-namespace mappings may be specified in any format, such as:
- A README file, or another form of human-readable documentation.
- A machine-readable format that is specific to a framework or other system the library is designed for. (e.g. composer.json)
- If the library is an extension for an existing framework, CMS or other system, then an _implicit_ mapping based on the extension name can be sufficient.

Recommended practice (optional):
- Every class should be at least in a two-level namespace, where the root namespace specifies the vendor, and the second-level namespace fragment specifies the package.
- An ideal simple one-package library has just one path-namespace mapping ($D, $N), where the directory $D is "src", and the namespace $N is the vendor plus package name.
- More generally, it is recommended that path-namespace mappings are specified for second-level namespaces.


### Instructions for autoloader developers

A universal PSR-X autoloader is a system that
- can be configured with any number of PSR-X path-namespace mappings.
- allows to register one or more callbacks on the spl autoload stack.

such that IF all the path-namespace mappings are valid PSR-X mappings, THEN whenever a fully qualified class name $C is being requested for autoloading,  
- IF there are one or more candidates ($D[i], $N[i], $F[i]) with
    - ($D[i], $N[i]) being one of the configured path-namespace mappings, and
    - ($D[i], $N[i], $F[i], $C) being a PSR-X match.

  THEN
    - Exactly ONE of the files $F[i] must be included.
      (Which candidate wins, is not subject of this spec)
    - (We assume that inclusion of this file will make $C available as a class or interface)
- On the other hand, if there is no such candidate, then no file must be included, and other class loaders on the spl autoload stack must get a chance to follow up.
- Either way, the class loader must not crash, or raise any errors or exceptions.


### Instructions for application developers

Instructions:
- Include any number of 3rd party libraries that claim to use PSR-X in your project.
- Choose a class loader that claims to support PSR-X.
- Understand the path-namespace mappings provided by the libraries, and the configuration interface provided by the class loader, and configure the class loader with the path-namespace mappings from the libraries.

Profit (if the libraries and the class loader keep their promises):
- The class loader will load all the classes provided by those libraries in PSR-X directories.
- The class loader will not crash.

(If the application uses any libraries or classes that are not organized in a PSR-X fashion, then it is the application developer's responsibility to find a suitable class loader for those classes.)
