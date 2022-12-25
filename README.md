# NAME
git-transpose - automatically and reliably transposes a range of commits onto another commit while mainining the tree-sameness of the final commit

# SYNOPSIS

    git transpose [--on-conflict [squash|split]] [--no-keep-editing|--keep-editing] [--update-head] [--onto to-commit] [[base-commit] commit]

# DESCRIPTION

## Transposing directly adjacent commits
Transposing two directly adjacent commits involves swapping their order.

Given:

$C^0_{n}=C^0_{n-2}{\leftarrow}P^0_{n-1}{\leftarrow}P^0_{n}$

transposition produces a new history, such that:

$C^1_{n}=C^0_{n-2}{\leftarrow}P^1_{n}{\leftarrow}P^1_{n-1}$ and $treesame( C^0_{n}, C^1_{n})$

If two commits edit different hunks, the swap can be achieved without conflict, otherwise the swap may induce a merge conflcit when the top commit is cherry-picked onto the base of the bottom commit.

Symbolically, a merge conflict and its resolution can be understood by decomposing $P^0_{n}$ into a part, $K^0_{n}$, that commutes with $P^0_{n-1}$ and a part, $N^0_{n}$, that doesn't.

$P^0_{n} = K^0_{n}{\leftarrow}N^0_{n}$

$C^0_{n}=C^0_{n-2}{\leftarrow}P^0_{n-1}{\leftarrow}K^0_{n}{\leftarrow}N^0_{n}$

Now we transpose $P^0_{n-1}$ and $K^0_{n}$ yieldng $C^1_{n}$ as follows:

$C^1_{n}=C^0_{n-2}{\leftarrow}K^0_{n}{\leftarrow}P^0_{n-1}{\leftarrow}N^0_{n}$

Next we define $P^1_{n-1}$ and $P^1_{n}$ as follows:

$P^1_{n}=K^0_{n}$

$P^1_{n-1}=P^0_{n-1}{\leftarrow}N^0_{n}$

yielding:

$C^1_{n}=C^0_{n-2}{\leftarrow}P^1_{n}{\leftarrow}P^1_{n-1}$ and $treesame( C^0_{n}, C^1_{n})$


## Transposing a single commit with a preceding range of commits

Transposing a commit with a non-adjacent commit can be achieved by iteratively transposing the commit onto parent commits until the iteratively transposed commit is directly adjacent to the originally non-adjacent commit at which point the two commits can be transposed according to the adjacent commit rules.

$C^1_{n}=C^0_{n-k-1}{\leftarrow}P^1_{n-k}{\leftarrow}P^1_{n-k+1}{\leftarrow}{\dots}{\leftarrow}P^1_{n}{\leftarrow}P^1_{n-1}$ and $treesame( C^0_{n}, C^1_{n})$

$\dots$

$C^k_{n}=C^0_{n-k-1}{\leftarrow}P^k_{n}{\leftarrow}P^k_{n-k}{\leftarrow}P^k_{n-k+1}{\leftarrow}{\dots}{\leftarrow}P^k_{n-1}$ and $treesame( C^0_{n}, C^k_{n})$

## Transposing a range of commits with a preceding range of commits

Transposing a range of j+1 commits onto the n-k-1'th commit can be done by repeating the non-adjacent transposition process for each of the j+1 commits in the range for a total of (k-j)*(j+1) transpositions.

$C^l_{n}=C^0_{n-k-1}{\leftarrow}P^l_{n-j}{\leftarrow}\dots{\leftarrow}P^l_{n}{\leftarrow}P^l_{n-k}{\leftarrow}P^l_{n-k+1}{\leftarrow}{\dots}{\leftarrow}P^l_{n-j-1}$ and $treesame( C^0_{n}, C^l_{n})$ and $l=(k-j)*(j+1)$

## Options

## [[base-commit] commit]

Specifies the range of commits to be transposed.

Defaults to `commit^1 commit`, if `commit` is specified or `HEAD^1 HEAD` if `commit` is not specified.

## --onto to-commit

The commit onto which the specified range of commits is to be transposed. Defaults to `base-commit^1`

## --on-conflict [squash|split]

When transposing directly adjacent commits, specifies whether the resolution of a merge conflict is squashed (squash) into the top transposed commit or is represented as an immediately following fixup! commit (split). Use of split implies --keep-empty.

## --no-keep-empty|--keep-empty

Elide empty commits from the rewritten history if *--no-keep-empty* is specified.

## --squash|--no-squash

Squashes the resulting rewritten to remove fixup commits.

## --update-head

Instead of outputing the results of the transposition, update the head to point to the transposed history. The updated HEAD commit will be tree-same with the original commit.

## Linear Reachability

For the purposes of this description, a commit A is linearly reachable from a commit B iff `git rev-list B ^A --merges` is empty and `git rev-list A ^B` is empty. `git transpose` will fail with a non-zero exit code if the specified commit and to-commit fail to satisfy the linear reachability requirements.

This command allows the user to transpose a specified range of commits that is linearly reachable from the current HEAD pointer onto another commit that is linearly reachable from the specified commit and to do so such that the commit referenced by the final HEAD pointer is tree-same to the commit referenced by the initial HEAD pointer. If this cannot be done, then the HEAD pointer is restored to its original value and the exit code is set to a non-zero value.

