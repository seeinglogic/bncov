# bncov - Scriptable Binary Ninja plugin for coverage analysis and visualization

bncov provides a scriptable interface for bringing together coverage
information with Binary Ninja's static analysis and visualization. Beyond
visualization, the abstractions in bncov allow for programmatic reasoning
about coverage. It was designed for interactive GUI use as well as for
factoring into larger analysis tasks and standalone scripts.

![Demo Overview](/pictures/demo_overview.gif)

This plugin is provided as a way to give back to the community,
and is not part of the Mayhem product.  If you're interested in Mayhem, the
combined symbolic execution and fuzzing system, check us out at
[forallsecure.com](http://forallsecure.com).

## CHANGELOG

Feb 2023: drcov format version 3 now supported.

Oct 2021: Some changes in the API:

- Added a `ctx` object that is keyed off the Binary View and helpers
  `bncov.get_ctx` and `bncov.get_covdb` to support multibinary use case in the
  UI, so now you can interactively use bncov across tabs!
- Changed old helpers `bncov.get_bv`/`bncov.get_covdb` to
  `bncov.make_bv`/`bncov.make_covdb` for making a Binary View from a target file
  and covdb from a binary and a coverage directory, respectively
- All function-related covdb member functions now default to keying off of
  function start addresses rather than names since function starts are unique
  and more usable for many applications. Extra optional args or helper functions
  implement the existing behavior.
- Abandoning Python2 suport
- Added `download_dynamorio.py` for the sloths
- Minor quality-of-life fixes

## Installation

The easiest way is to install via the Binary Ninja plugin manager!
The only difference when installing via plugin manager is that wherever
you see `import bncov`, you'll do `import ForAllSecure_bncov as bncov`.

Alternatively:

 - Clone or copy this directory into your binja plugins folder.
([More detailed instructions here](https://docs.binary.ninja/guide/plugins/index.html#using-plugins))
 - (Optional) pip install msgpack if you want to enable loading/saving
coverage database files.

## Usage

Check out the [tutorial](/tutorial/) for a complete walkthrough or how to get
started right away using data that's already included in this repo!

First collect coverage information in DynamoRIO's drcov format
([example script](/dr_block_coverage.py)).

To use in Binary Ninja GUI:

1. Open the target binary, then import coverage files using one of
the commands in `bncov/Coverage Data/Import *`
either from the Tools menu or from the context (right-click) menu.
2. Explore the coverage visualization and explore additional analyses from
the right-click menu or with the built-in interpreter and `import bncov`
followed by `covdb = bncov.get_covdb(bv)`.

Scripting:

1. Ensure bncov's parent directory is in your module search path
OR add it to sys.path at the top of your script like this:
`sys.path.append(os.path.split(os.path.normpath('/path/to/bncov'))[0])`
2. `import bncov` and write scripts with the CoverageDB class in
`coverage.py`, check out the `scripts` folder for examples.

## Screenshots

Import a coverage directory containing trace files to see blocks colored in
heat map fashion: blocks covered by most traces (blue) or by few traces
(red). Additional context commands (right-click menu) include frontier
highlighting and a per-function block coverage report.

* Watch a directory to have new coverage results get automatically highlighted
when new coverage files appear

![Watch Coverage Directory](/pictures/Coverage-watching.gif)

* See at a glance which blocks are only covered by one or a few traces
(redder=rarer, bluer=more common)

![See Relative Rarity](/pictures/Relative-Rarity.png)

* Quickly discover rare functionality visually or with scripting

![Highlight Rare Blocks](/pictures/Heartbleed-Rare-block.png)

* Identify which blocks have outgoing edges not covered in the traces

![Highlight Frontier Blocks](/pictures/Frontier-Highlight.png)

* See coverage reports on functions of interest or what functionality may not
be hit, or write your own analyses for headless scripting.

![Block Coverage Report](/pictures/Coverage-Report.png)

## Notes

Currently the plugin only deals with block coverage and ingests files in the
drcov format or "module+offset" format. Included in the repo is
`dr_block_coverage.py` which can be used for generating coverage files, just
specify your DynamoRIO install location with an environment variable (or
modify the script) and it can process a directory of inputs. DynamoRIO binary
packages can be found
[here](https://github.com/DynamoRIO/dynamorio/wiki/Downloads) or you can use the
included `download_dynamorio.py` script. See the
[tutorial](/tutorial/) for a complete walkthrough.

Please file any feature requests/bugs as issues on GitHub, we welcome any input
or feedback.

## Scripting

bncov was designed so users can interact directly with the data structures
the plugin uses. See the `scripts/` directory for more ideas.

* Helpful CoverageDB members:
    * trace_dict (maps filenames to set of basic block start addresses)
    * block_dict (maps basic block start addresses to files containing it)
    * total_coverage (set of start addresses of the basic blocks covered)

* Helpful CoverageDB functions:
    * get_traces_from_block(addr) - get files that cover the basic block
      starting at addr.
    * get_rare_blocks(threshold) - get blocks covered by <= 'threshold' traces
    * get_frontier() - get blocks that have outgoing edges that aren't covered
    * get_functions_from_blocks(blocks, by_name=False) - return dict mapping
      function starts/names to blocks they contain
    * get_traces_from_function(function_start) - return set of traces that have
      coverage in the specified function
    * get_traces_from_function_name(function_name, demangle=False) - return set
      of traces that have coverage in the specified function

* You can use Binary Ninja's python console and built-in python set operations with
bncov.highlight_set() to do custom highlights in the Binary Ninja UI.
