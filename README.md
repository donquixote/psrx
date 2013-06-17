psrx
====

Alternative proposal for PSR-X autoloader spec (under construction)



## The Spec

### Goal

The spec deals with three questions:
- How to write a PHP library, so that it can be made to work with any PSR-X compliant class loader.
- How to write a class loader, so that it can be made to work with any PSR-X compliant PHP library.
- How can a PSR-X compliant library be made to work with a PSR-X compliant class loader.


### Definition: PSR-X match

Let
- $D be a filesystem directory.
- $N be a valid PHP namespace.
- $F be a valid file path.
- $C be a valid fully-qualified PHP class name.

Then ($D, $N, $F, $C) is a PSR-X match, IF there are two strings $path_suffix and $class_suffix, such that:
- $class_suffix is the string that results from replacing every directory separator in $dir_suffix with an underscore.
- $F = $D + '/' + $path_suffix + '.php'.
- $F is an existing file.
- $C = $N + '\' + $class_suffix.


### Definition: PSR-X path-namespace mapping

Let $D be a filesystem directory, $N be a PHP namespace.  
($D, $N) is a PSR-X path-namespace mapping, IF  
For every file $F,  
if there is a fully-qualified class name $C,  
such that ($D, $N, $F, $C) is a PSR-X match,  
then the inclusion of $F from a PHP script, at a time when the class $C is not already defined,
- must make $C available as a class or interface.
- (must not have any other side effects (under discussion))
- must NOT cause the script to crash, exit, or raise any errors or exceptions.


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
