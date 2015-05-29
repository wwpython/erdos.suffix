# erdos.suffixtree

A Clojure library designed to ... well, that part is up to you.



## Usage

1. install a Java environment and [Leiningen](https://leiningen.org/)

2. clone source code to your computer using git: `git clone https://github.com/erdos/erdos.suffix`

3. run the demo application `lein run /path/to/word/list.txt`

It reads a newline separated list of words from a file and then repeatedly prompts the user for substring search. Can also be used in an unix manner: `echo "resz" | lein run /path/to/word/list.txt`


## API - packages

### erdos.suffixtrie

This package contains a suffix trie agorithm. Construction is O(N^2) time, space is O(N^2) time.

`(->trie "word1" "word2" ...)` Create a suffix trie object.

`(suffix-of trie "abc")` Returns a lazy list of words having the suffix "abc"

`(find-words trie "abc")` Returns a lazy list of words that contain "abc".

### erdos.suffixtree

Suffix tree algorithm. Construction is based on Ukkonen's algorithm, thus providing a very fast O(N) construction time and requires O(N) space.

`(->tree "long text")` Creates a suffix tree object for a single long text. 

`(index-of triee "substring")` Returns the integer index of a substring in a tree, or `nil` when not found.

### erdos.suffixarray

Suffix array algorithm.

`(->suffixarray "long text")` Constructs a suffix array.

`(index-of suffixarray "subs")` Returns integer index of substring in suffix array or `nil` when not found.

### erdos.str

String and lazy sequence manipulation.

`(prefixes xs)` Creates a sequence of all prefixes of xs.

`(suffixes xs)` Creates a seq of all sufixes.

`(subseqs xs)` Creates seq of all subsequences. That is a list of all prefixes of all suffixes.

## Resources

 - Generalized Suffix Tree implementation in java [1](https://github.com/abahgat/suffixtree) and [2](https://gist.github.com/bicepjai/3355993)
 - GST in C [3](https://github.com/Rerito/suffix-tree)
 - ST in ruby [4](https://gist.github.com/suchitpuri/9304856)
 - Nice introduction to ST [5](http://www.cise.ufl.edu/~sahni/dsaaj/enrich/c16/suffix.htm)
 - [Ukkonen's article](https://www.cs.helsinki.fi/u/ukkonen/SuffixT1withFigs.pdf)

## Results

- found a [bug in a python implementation](https://github.com/zhangliyong/generalized-suffix-tree/issues/1)

## License

Copyright © 2015 FIXME

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.
