This variation is even closer to the original proposal.

It specifies the behavior of a class loader, without mentioning the state of the filesystem.

Compared to the original proposal by Paul, it attempts to
- avoid loopholes and be crystal clear.
- avoid any statements that are not within the responsibility of the autoloader.

Especially, it does no longer talk about libraries or applications, it only talks about the autoloader. But it does so in a way that instructions for libraries can be easily derived.

The goal is to be equivalent with the intended meaning of Paul's proposal, or whatever the group agrees on, but be more precise in the wording.

If any part of it conflicts with the intended meaning the group has in mind, then it can be changed.


## Summary

A PSR-X autoloader is a callback registered on the spl autoload stack, that operates with an ordered list (hardcoded or configurable) of path-namespace mappings.
Each mapping consists of a filesystem directory and a PHP namespace.

Whenever the autoload callback is triggered, it has to look for matching files within the registered directories, and include one of them.


## Definitions

### Definition: Path-namespace mapping

A path-namespace mapping is an existing filesystem directory together with a valid PHP namespace.

(Todo: Talk about trailing namespace/directory separators)

We refer to "directory tree" as the collection of the directory itself, and all subfolders at any depth.


### Definition: PSR-X Match

For a filesystem directory and a PHP namespace (that we call the "path-namespace mapping"), a file within the directory tree, and a fully-qualified class name, we say:

The file "matches" the fully qualified class name with respect to the path-namespace mapping, IF  
- The class can be split up into a namespace prefix and a relative class name,
- The file path can be split up into a base directory and a relative file path,

such that
- The namespace prefix is the namespace from the mapping.
- The base directory is the directory from the mapping.
- The relative file path can be built from the relative class name, by replacing every namespace separator with a directory separator, and appending ".php".


### Definition: PSR-X autoloader

A PSR-X autoloader is a callback registered on the spl autoload stack, that operates on a (hardcoded or configurable) *ordered list* of path-namespace mappings.

Whenever the autoloader is triggered with a fully qualified class name, it has to:
- If one or more files exist that "match" the fully qualified class name with respect to one of the mappings, then it must include or require *exactly one* of those files.
- If more than one of the mappings has a matching file, then it must choose a/the file from the mapping that is first in the list.
- If no match is found, then it must do nothing.
- Either way, it may not crash, raise errors of any level, or throw any exceptions *by itself*.

Note: If the autoloader is triggered with an invalid class name, then it is the implementor's choice what to do. The autoloader may simply crash, the spec does not care.

Note: (*) It can be shown that for one mapping and one fully qualified class name, no more than one file can be a "match".

Note: If the script crashes during inclusion of the file, this is not the responsibility of the autoloader.

Note: The file is supposed to define the class that is expected. Whether it does so or not, is not the responsibility of the autoloader. The autoloader will move on *as if* the class was successfully defined.


## Appendix: Instructions developers that use a PSR-X autoloader.

The spec explains how the class loader has to be *implemented*, but not how it has to be *used* so it works reliably.

This appendix clarifies on these issues.


### Definition: PSR-X path-namespace Mapping

A path-namespace mapping qualifies as "PSR-X", IF  
For every fully-qualified class name, if there is a file in the directory tree that "matches" the class with respect to the mapping,
then the inclusion of this file from a PHP script MUST make a class or interface available with the given fully-qualified class name.

(Todo: More criteria can be discussed)


### Definition: PSR-X mapping conflict

For two PSR-X path-namespace mappings, we say these mappings are in conflict, if there is a file, such that
- The file is in the directory tree of each mapping.
- There is a class that "matches" the file with respect to the first mapping.
- There is another class that "matches" the file with respect to the second mapping.
- These two classes are different.

For two PSR-X path-namespace mappings, we say these mappings are in a *potential* conflict, if a file *could be created* such that
- The file would be in the directory tree of each mapping.
- Each mapping would still be PSR-X.
- The two mappings would be in conflict, if the file is added.

Note: The second part is necessary to extend the definition to empty directories.


### Criteria for the autoloader's path-namespace mappings

It was said in the spec that the autoloader "operates with" an ordered list of path-namespace mappings, which may be hardcoded or configurable.

If
- each of the mappings is a PSR-X mapping.
- no two of the mappings are in a potential conflict.

then the autoloader will work correctly, that is, it will successfully load the classes and interfaces under the registered directories, and it will not crash.

Note: If the above is not the case, then the autoloader may still work successfully, but we do not guarantee for it.

Note: If the path-namespace mappings are configurable, then it is the implementor's choice whether to check if the registered mappings are valid PSR-X mappings, or if this should be taken for granted.
