This variation identifies the goals and audience in a different way, trying to be closer to the original proposal.

Especially, it does no longer talk about libraries or applications, it only talks about the autoloader. But it does so in a way that instructions for libraries can be easily derived.

The goal is to be equivalent with the intended meaning of Paul's proposal, or whatever the group agrees on, but be more precise in the wording.

If any part of it conflicts with the intended meaning the group has in mind, then it can be changed.


## Summary

A PSR-X autoloader is a callback registered on the spl autoload stack, that operates with an ordered list (hardcoded or configurable) of path-namespace mappings.
Each mapping consists of a filesystem directory and a PHP namespace.

If those mappings, and the files within the mapped directories, follow certain criteria, then the autoloader must be able to load the classes defined in these directories.


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


### Definition: PSR-X path-namespace Mapping

A path-namespace mapping is qualifies as "PSR-X", if
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


### Definition: PSR-X autoloader

A PSR-X autoloader is a callback registered on the spl autoload stack, that operates on a (hardcoded or configurable) *ordered list* of path-namespace mappings, where
- each of the mappings is a PSR-X mapping.
- no two of the mappings are in a potential conflict.

Whenever the autoloader is triggered with a fully qualified class name, it has to:
- If one or more files exist that "match" the fully qualified class name with respect to one of the mappings, then it must include or require *exactly one* of those files.
- If more than one of the mappings has a matching file, then it must choose a/the file from the mapping that is first in the list.
- If no match is found, then it must do nothing.
- Either way, it may not crash, raise errors of any level, or throw any exceptions.

Note: If the autoloader is triggered with an invalid class name, then it is the implementor's choice what to do. The autoloader may simply crash, the spec does not care.

Note: If the path-namespace mappings are configurable, then it is the implementor's choice whether to check if the registered mappings are valid PSR-X mappings.

Note: (*) It can be shown that for one mapping and one fully qualified class name, no more than one file can be a "match".
