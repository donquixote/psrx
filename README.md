psrx
====

Alternative proposal for PSR-X autoloader spec



## The Spec

Let $D be a filesystem directory, $N be a PHP namespace.
($D, $N) is a PSR-X path-namespace mapping, IF
- For every file $F = $D + '/' + $s + '.php' in the directory, if $C = $N + '\' + §s is a valid PHP class name, then the inclusion of $F by a PHP script must make the class $C available.
- For every subdirectory $DD = $D + '/' + $s, if $NN = $N + '\' + $s is a valid PHP namespace name, then ($DD, $NN) must be a PSR-X namespace mapping.
- $D has a finite depth (in practice this means it has no loopy symlinks)

Note: This recursive definition is well-defined, because of the finite depth restriction.


A *PHP library* is PSR-X compliant, if it provides (*) a number of path-namespace mappings, such that
- every such mapping ($D, $N) is a PSR-X path-namespace mapping.
- the directories are within the library
- each class of the library is in one of those directories.

