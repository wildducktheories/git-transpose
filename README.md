# NAME
git-transpose - automatically and reliably transposes a linear range of commits onto another commit while maintaining the tree-sameness of the final commit in the revised commit history.

# SYNOPSIS

    git transpose [--on-conflict [squash|split]] [--no-keep-empty|--keep-empty] [--update-head] [--onto to-commit] [[base-commit] commit]

# DESCRIPTION

# OPTIONS

## [[base-commit] commit]

Specifies the range of commits to be transposed.

The range includes `commit` but does not include `base-commit` which is the boundary commit at the base of the range to be transposed.

`commit` defaults to HEAD, if not specified. `base-commit` defaults to `commit^1`, if not specified.

`base-commit` must be linearly reachable from `commit`.

If **--update-head** is specified, `commit` must be linearly reachable from HEAD.

## --onto to-commit

The commit onto which the specified range of commits is to be transposed. Defaults to `base-commit^1`

`to-commit` must be linearly reachable from `base-commit`

## --on-conflict [squash|split]

When transposing directly adjacent commits, specifies whether the resolution of a merge conflict is squashed ('squash') into the top transposed commit or is represented as an immediately following **fixup!** commit (`split`).

Use of split implies **--keep-empty**. The default is `squash`.

## --no-keep-empty|--keep-empty

If **--no-keep-empty** is specified empty commits are elieded from the rewritten history, otherwise if **--keep-empty** is specified they are preserved. If **on-conflict split** is specified, then **--keep-empty** is implied.

## --update-head

If **--update-head** is specified, then HEAD is replaced with a revised commit history such that the updated HEAD is tree-same to the original HEAD and points to the revised `base-commit` while the range of commits `commit ^base-commit` has been transposed
onto `to-commit`.

It is an error to specify **--update-head** unless `commit` is linearly reachable from `HEAD`.

# OUTPUT

If **--update-head** is specified, no output is generated on stdout.

Otherwise, two SHA1 hashes are written to stdout on a single line separated by a space.

The first SHA1 identifies the commit in the revised commit history corresponding to the specified `commit` argument in the original commit history.

The second SHA1 identifies the commit in the revised commit history that corresponds to the explicitly or implicitly specified `base-commit` argument in the original commit history. The rewritten `base-commit` will always be tree-same to the original `commit` argument.


# THEORY

## Transposing directly adjacent commits
Transposing two directly adjacent commits involves swapping their order.

Given:

$T^0_{n}=T^0_{n-2}{\leftarrow}P^0_{n-1}{\leftarrow}P^0_{n}$

transposition produces a new history, such that:

$T^1_{n}=T^0_{n-2}{\leftarrow}P^1_{n}{\leftarrow}P^1_{n-1}$ and $T^0_{n} = T^1_{n}$

If two commits edit different hunks, the swap can be achieved without conflict, otherwise the swap may induce a merge conflcit when the top commit is cherry-picked onto the base of the bottom commit.

Symbolically, a merge conflict and its resolution can be understood by decomposing $P^0_{n}$ into a part, $K^0_{n}$, that commutes with $P^0_{n-1}$ and a part, $N^0_{n}$, that doesn't.

$P^0_{n} = K^0_{n}{\leftarrow}N^0_{n}$

$T^0_{n}=T^0_{n-2}{\leftarrow}P^0_{n-1}{\leftarrow}K^0_{n}{\leftarrow}N^0_{n}$

Now we transpose $P^0_{n-1}$ and $K^0_{n}$ yieldng $T^1_{n}$ as follows:

$T^1_{n}=T^0_{n-2}{\leftarrow}K^0_{n}{\leftarrow}P^0_{n-1}{\leftarrow}N^0_{n}$

Next we define $P^1_{n-1}$ and $P^1_{n}$ as follows:

$P^1_{n}=K^0_{n}$

$P^1_{n-1}=P^0_{n-1}{\leftarrow}N^0_{n}$

yielding:

$T^1_{n}=T^0_{n-2}{\leftarrow}P^1_{n}{\leftarrow}P^1_{n-1}$ and $T^0_{n} = T^1_{n}$

## Transposing a single commit with a preceding range of commits

Transposing a commit with a non-adjacent commit can be achieved by iteratively transposing the commit onto parent commits until the iteratively transposed commit is directly adjacent to the originally non-adjacent commit at which point the two commits can be transposed according to the adjacent commit rules.

$T^1_{n}=T^0_{n-k-1}{\leftarrow}P^1_{n-k}{\leftarrow}P^1_{n-k+1}{\leftarrow}{\dots}{\leftarrow}P^1_{n}{\leftarrow}P^1_{n-1}$ and $T^0_{n}=T^1_{n}$

$\dots$

$T^k_{n}=T^0_{n-k-1}{\leftarrow}P^k_{n}{\leftarrow}P^k_{n-k}{\leftarrow}P^k_{n-k+1}{\leftarrow}{\dots}{\leftarrow}P^k_{n-1}$ and $T^0_{n}=T^k_{n}$

## Transposing a range of commits with a preceding range of commits

Transposing a range of j+1 commits onto the n-k-1'th commit can be done by repeating the non-adjacent transposition process for each of the j+1 commits in the range for a total of (k-j)*(j+1) transpositions.

$T^l_{n}=T^0_{n-k-1}{\leftarrow}P^l_{n-j}{\leftarrow}\dots{\leftarrow}P^l_{n}{\leftarrow}P^l_{n-k}{\leftarrow}P^l_{n-k+1}{\leftarrow}{\dots}{\leftarrow}P^l_{n-j-1}$ and $T^0_{n} = T^l_{n}$ and $l=(k-j)*(j+1)$

# Linear Reachability

For the purposes of this description, a commit A is linearly reachable from a commit B iff `git rev-list B ^A --merges` is empty and `git rev-list A ^B` is empty. Another way of saying this is that there is only one path in the commit history from
A to B.
