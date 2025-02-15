# haskell-template

Get a Haskell development environment up and running quickly, as long as Nix is available on your system[^windows].

[^windows]: On Windows, you may install Nix [under WSL2](https://nixos.wiki/wiki/Nix_Installation_Guide#Windows_Subsystem_for_Linux_.28WSL.29) and use the [Remote - WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) to connect from the native VSCode. This runs your project under Linux while providing a near-native development experience on Windows.

This repository is a Haskell project template that is optimized for a fully reproducible and friendly development environment. It is based on:

- [Nix](https://srid.ca/haskell-nix) + [Flakes](https://serokell.io/blog/practical-nix-flakes) (via [`github:srid/haskell-flake`](https://github.com/srid/haskell-flake)) + GHC 9
- VSCode + [HLS](https://github.com/haskell/haskell-language-server)
- [fourmolu](https://github.com/fourmolu/fourmolu) autoformatting 
- [Relude](https://github.com/kowainik/relude#relude) as Prelude.
  - `.hlint.yaml` is [from relude](https://github.com/kowainik/relude/blob/main/.hlint.yaml)

## Getting Started

First-time setup:

- [Install Nix](https://nixos.org/download.html) (>= 2.8) & [enable Flakes](https://nixos.wiki/wiki/Flakes) (Using Windows? See footnote[^windows])
- Run `nix develop -i -c haskell-language-server` to sanity check your environment 
- [Open as single-folder workspace](https://code.visualstudio.com/docs/editor/workspaces#_singlefolder-workspaces) in Visual Studio Code
    - When prompted by VSCode, install the [workspace recommended](https://code.visualstudio.com/docs/editor/extension-marketplace#_workspace-recommended-extensions) extensions
    - <kbd>Ctrl+Shift+P</kbd> to run command "Nix-Env: Select Environment" and then select `shell.nix`. 
        - The extension will ask you to reload VSCode at the end. Do it.

To run the program with auto-recompile:

- Press <kbd>Ctrl+Shift+B</kbd> in VSCode, or run `bin/run` in terminal, to launch Ghcid running your program.

Open `Main.hs`, and expect all HLS IDE features like hover-over tooltip to work out of the box. Try changing the source, and expect Ghcid to re-compile and re-run the app in the terminal below.

---

Renaming the project:

```sh
# First, click the green "Use this template" button on GitHub to create your copy.
git clone <your-clone-url>
cd your-project
NAME=myproject

git mv haskell-template.cabal ${NAME}.cabal
nix run nixpkgs#sd -- haskell-template ${NAME} * */*
git add . && git commit -m rename
```

## Tips

- Run `nix flake update` to update all flake inputs.
- Run `nix --option sandbox false build .#check -L` to run the flake checks.
- Run `treefmt` in nix shell to autoformat the project. This uses [treefmt](https://github.com/numtide/treefmt), which uses `./treefmt.toml` (where fourmolu and nixpkgs-fmt are specified).
- Run `bin/hoogle` to start Hoogle with packages in your cabal file.
- Run `bin/test` to run the test suite.
- Run the application without installing: `nix run github:srid/haskell-template` (or `nix run .` from checkout)

## Common workflows

### Adding tests 

1. Split any logic code out of `Main.hs` into, say, a `Lib.hs`.
1. Correspondingly, add `other-modules: Lib` to the "shared" section of your cabal file.
1. Add `tests/Spec.hs` (example below):
    ```haskell
    module Main where

    import Lib qualified
    import Test.Hspec (describe, hspec, it, shouldContain)

    main :: IO ()
    main = hspec $ do
      describe "Lib.hello" $ do
        it "contains the world emoji" $ do
          toString Lib.hello `shouldContain` "🌎"
    ```
1. Add the tests stanza to the cabal file:
    ```cabal
    test-suite tests
    import:         shared
    main-is:        Spec.hs
    type:           exitcode-stdio-1.0
    hs-source-dirs: tests
    build-depends:  hspec
    ```
1. Update `hie.yaml` accordingly; for example, by adding,
    ```yaml
      - path: "tests"
        component: "test:tests"
    ```
1. Add `bin/test` and `chmod a+x` it:
    ```sh
    #!/usr/bin/env bash
    set -xe

    exec nix develop -i -c ghcid -c "cabal repl test:tests" -T :main
    ```
1. Commit your changes to Git, and test it out by running `bin/test`.

## Discussions

Got questions? Ideas? Suggestions? Post them here: https://github.com/srid/haskell-template/discussions
