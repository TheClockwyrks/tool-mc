# history

The record of how this repository's history was extracted from the monorepo,
[The Test Cabinet](https://github.com/TheClockwyrks/TheTestCabinet), where the
crates here lived until the extraction. Nothing builds or tests against these
files; they are kept so the provenance outlives the scratch clone the
extraction ran in.

## The extraction

The history below the scaffold commit is the monorepo's `issue/repos-4` at
`25923d22c5ec8eabf96f1b97f21a546c1ac3db25`, the branch whose commit
`77c83390e` moved the crates' model-core, voxel-mesh and model-skin
dependencies onto the tool-model-core repository, filtered to the paths in
[`paths.txt`](paths.txt) by git filter-repo 2.47.0:

```sh
git clone -q --no-local --single-branch --branch issue/repos-4 --no-tags \
  <the-test-cabinet> x
cd x
uvx --from git-filter-repo==2.47.0 git-filter-repo --force \
  --paths-from-file ../paths.txt --prune-empty always --preserve-commit-hashes
```

Of the 6,976 commits reachable from that commit, 21 touched a path in the list
and were kept (1 of them a merge); the rest were pruned. No tag was carried.
The tip tree of every moved directory hashes the same as the monorepo's at that
commit:

```text
crates/mc/       e147974dfac5aafe390353bc4e759009246f8596
crates/mc-anim/  bc111abbd7f986b316c391d7a88a817fdd028abb
crates/mc-skin/  7befc7a6944d3025f353b5955241818c5c1bb17e
```

The paths are kept where they were in the monorepo, so the filter needed no
rename and the path dependency of `mc-anim` on `mc` works as it did.

## The files

- [`paths.txt`](paths.txt) is the list handed to `--paths-from-file`
  (sha256 `e621f4885109eb4cdd9dbcf808b14aeb7d1ff165f8bc8917fec74c36dacac3ba`).
  It is generated, not hand-written: the three moved directories
  (`crates/mc/`, `crates/mc-anim/`, `crates/mc-skin/`), and every path a
  tracked file under them had before a rename or copy, collected by
  `git log --follow -M -C --name-status` over each of the 8 files. Two copies
  were found: `crates/mc/src/bin/mc.rs` began as a copy of
  `crates/voxel/src/bin/voxel.rs`, and `crates/mc-anim/src/main.rs` as a copy
  of `crates/voxel-anim/src/main.rs`, which was itself renamed from
  `crates/voxel/src/bin/voxel-anim.rs`. Those three paths are in the list.
- [`commit-map`](commit-map) is filter-repo's map from each monorepo commit (left)
  to the commit it became here (right), all zeros for a pruned one.
  `--preserve-commit-hashes` left the commit ids quoted in the messages as they
  were, so an id in a message below the scaffold commit names a monorepo commit,
  and this map translates it.

## The live-path review

`paths.txt` intersected with the monorepo's tree at the extraction commit names,
outside the three moved directories, exactly two live files:

```text
crates/voxel/src/bin/voxel.rs
crates/voxel-anim/src/main.rs
```

Each is a copy source: an mc file began as a copy of it, and the voxel crates
kept their own, which now live in the tool-voxel repository. They were kept in
the list so `git log --follow` and blame of the files copied from them reach
their history before the copy. Their commits after the copy, which touch the
voxel binaries only, are in this history too, and the scaffold commit deletes
the two files. The third path, `crates/voxel/src/bin/voxel-anim.rs`, was
already gone from the monorepo's tree.
