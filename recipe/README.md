# FastJet (C++) packaging on conda-forge

This feedstock supplies three related packages:

- **`fastjet-cxx`** – the runtime shared libraries `libfastjet`, `libfastjettools` and
  `libfastjetplugins` (`fastjet.dll`, `fastjettools.dll` and `fastjetplugins.dll` on Windows).
  Nothing else: no headers, no `fastjet-config`, no CMake files.
- **`fastjet-cxx-devel`** – everything needed to compile and link against FastJet:
  - the headers under `include/fastjet/`;
  - `fastjet-config`;
  - the CMake package configuration (`find_package(fastjet)`, targets `fastjet::fastjet`,
    `fastjet::fastjettools` and `fastjet::fastjetplugins`);
  - the import libraries on Windows.

  It depends on the exactly matching `fastjet-cxx` build, and on `siscone-devel` and `cgal-cpp`,
  because `fastjetConfig.cmake` calls `find_package(siscone REQUIRED)` and `find_package(CGAL)`.
- **`fastjet-cxx-python`** – FastJet's own SWIG Python bindings, imported as `fastjet_cxx`.
  These are not the scikit-hep `fastjet` Python package, which comes from the
  [fastjet feedstock](https://github.com/conda-forge/fastjet-feedstock).

`fastjet-cxx` and `fastjet-cxx-devel` carry `run_exports` on `fastjet-cxx` (`x.x`), so anything
built against FastJet gets the runtime package as a run dependency automatically.

## How recipes use these packages

### A. The recipe compiles or links against FastJet

Put `fastjet-cxx-devel` in `host`. The runtime dependency on `fastjet-cxx` comes from its
`run_exports`; do not list it in `run`.

```yaml
# recipe.yaml (excerpt)
requirements:
  host:
    - fastjet-cxx-devel
```

### B. The package compiles code against FastJet at run time, or its installed headers include FastJet headers

Some packages compile user code when they run, for example event generators or analysis frameworks
that build plugins against their own headers, or tools that call `fastjet-config`. If those builds
or headers need FastJet, also put `fastjet-cxx-devel` in `run`. The same applies to the `run`
requirements of your own `*-devel` output if its installed headers or CMake configuration reference
FastJet.

```yaml
# recipe.yaml (excerpt)
requirements:
  host:
    - fastjet-cxx-devel
  run:
    - fastjet-cxx-devel
```

### C. The package only runs software that was built against FastJet

Nothing to do: the `run_exports` above already pull in `fastjet-cxx`.

### D. Interactive development

To compile your own code against FastJet in an environment, install `fastjet-cxx-devel` (plus a
compiler, e.g. `cxx-compiler`). `fastjet-config --pythonpath` prints a path only when
`fastjet-cxx-python` is installed too.

## Further details

- Before the split (`fastjet-cxx` 3.5.1 build 5 and earlier), `fastjet-cxx` held the libraries, the
  headers, `fastjet-config` and the CMake files together. From 3.5.1 build 6 on, everything but the
  libraries is only in `fastjet-cxx-devel`. A recipe that kept `fastjet-cxx` in `host` now fails at
  `find_package(fastjet)` or when it compiles against FastJet's headers: switch it to
  `fastjet-cxx-devel` (case A).
- SISCone and fjcontrib are split the same way: `siscone` / `siscone-devel` and `fastjet-contrib` /
  `fastjet-contrib-devel`. Each feedstock's `recipe/README.md` describes its packages.
- See conda-forge/fastjet-cxx-feedstock#27 for the split.
