# NAME
git-transpose - a command to reliably transposes a range of commits onto another commit and restore the following commits onto the transposed range so that the final commit is tree-same to the starting commit. 

# SYNOPSIS

    git transpose [ [--onto to-commit] [--on-conflict [abort|stop|edit]] [from-commit [commit]] | --continue | --abort ]

# DESCRIPTION

Transposing two adjacent commits involves swapping their order. If two commits edit different hunks, the swap can be achieved without conflict, otherwise the swap may induce a merge conflcit in one or both of the commits. However, if the user is able to resolve the first conflict, then the second conflict can be resolved automatically by using the tree of the original final commit as a reference for the tree of the transposed final commit.

$C^0_{n}=C^0_{n-2}{\leftarrow}P^0_{n-1}{\leftarrow}P^0_{n}$

After transposition:

$C^1_{n}=C^0_{n-2}{\leftarrow}P^1_{n}{\leftarrow}P^1_{n-1}$ and $treesame( C^0_{n}, C^1_{n})$

Transposing a commit with a non-adjacent commit can be achieved done by iteratively transposing the commit onto parent commits until the iteratively transposed commit is directly adjacent to the non-adjacent commit at which point the two commits can be transposed according to the adjacent commit rules.

$C^1_{n}=C^0_{n-k-1}{\leftarrow}P^1_{n-k}{\leftarrow}P^1_{n-k+1}{\leftarrow}{\dots}{\leftarrow}P^1_{n}{\leftarrow}P^1_{n-1}$ and $treesame( C^0_{n}, C^1_{n})$

$\dots$

$C^k_{n}=C^0_{n-k-1}{\leftarrow}P^k_{n}{\leftarrow}P^k_{n-k}{\leftarrow}P^k_{n-k+1}{\leftarrow}{\dots}{\leftarrow}P^k_{n-1}$ and $treesame( C^0_{n}, C^k_{n})$

Transposing a range of j+1 commits onto the n-k-1'th commit can be done by repeating the non-adjacent transposition process for each of the j+1 commits in the range for a total of (k-j)*(j+1) transpositions.

$C^l_{n}=C^0_{n-k-1}{\leftarrow}P^l_{n-j}{\leftarrow}\dots{\leftarrow}P^l_{n}{\leftarrow}P^l_{n-k}{\leftarrow}P^l_{n-k+1}{\leftarrow}{\dots}{\leftarrow}P^l_{n-j-1}$ and $treesame( C^0_{n}, C^l_{n})$ and $l=(k-j)*(j+1)$

## Linear Reachability

For the purposes of this description, a commit A is linearly reachable from a commit B iff `git rev-list B ^A --merges` is empty and `git rev-list A ^B` is empty. `git transpose` will fail with a non-zero exit code if the specified commit and to-commit fail to satisfy the linear reachability requirements.

This command allows the user to transpose a specified range of commits that is linearly reachable from the current HEAD pointer onto another commit that is linearly reachable from the specified commit and to do so such that the commit referenced by the final HEAD pointer is tree-same to the commit referenced by the initial HEAD pointer. If this cannot be done, then the HEAD pointer is restored to its original value and the exit code is set to a non-zero value.

# EXIT CODES

An exit code of 0 is set iff:

- the specified (or implied) commit is linearly reachable from the current HEAD pointer
- the specified (or implied) to-commit is linearly reachable from the specified (or implied) commit
- the specified commit was successfully transposed onto the specified (or implied to-commit) and the final commit referred to by the HEAD pointer is tree-same to the commit specified original HEAD pointer

An exit code of 127 is set iff:

- an on-conflIct action of stop was specified
- the specified (or implied) commit is linearly reachable from the current HEAD pointer
- the specified (or implied) to-commit is linearly reachable from the specified (or implied) commit
- the specified commit was successfully transposed onto a commit reachable from the specified commit, but that commit was not the specifed (or implied) to-commit.

An exit code of 126 is set iff:

- a on-conflict action of edit was specified
- the cherry-pick of the specified (or implied) commit onto the commit referenced by the current HEAD pointer failed with a merge conflict
- the transpose operation is still in progress

This code can also be specified if git transpose is invoked with a --continue option but the working tree is dirty or the HEAD pointer has been modified.

An exit code of 125 is set iff:

- a previously started transpose operation was aborted either implicitly (via an --on-conflict action of abort) or by use of an an explicit --abort option

In this case, the commit referenced by the HEAD pointer is identical to its value at the time the transpose operation was started.





