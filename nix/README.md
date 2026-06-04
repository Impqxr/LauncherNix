# PineconeMC Nix Packaging

## Installing a development release (flake)

We use [Cachix](https://cachix.org/) to cache PineconeMC builds.
If you want to avoid rebuilds you may add Cachix to your substitutors, or use `--accept-flake-config`
to temporarily enable it when using `nix` commands.

Example (NixOS):

```nix
{
  nix.settings = {
    trusted-substituters = [ "https://pineconemc-unofficial.cachix.org" ];

    trusted-public-keys = [
      "pineconemc-unofficial.cachix.org-1:aFVxmb1DKgTEdeaCUATWQXz73nm6oPI2r+Vd+wfgR/0="
    ];
  };
}
```

### Installing the package directly

After adding `github:Impqxr/LauncherNix` to your flake inputs, you can access the flake's `packages` output.

Example:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

    pineconemc = {
      url = "github:Impqxr/LauncherNix";

      # Optional: Override the nixpkgs input of pineconemc to use the same revision as the rest of your flake
      # Note that this may break the reproducibility mentioned above, and you might not be able to access the binary cache
      #
      # inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs =
    { nixpkgs, pineconemc, ... }:
    {
      nixosConfigurations.foo = nixpkgs.lib.nixosSystem {
        modules = [
          ./configuration.nix

          (
            { pkgs, ... }:
            {
              environment.systemPackages = [ pineconemc.packages.${pkgs.system}.pineconemc ];
            }
          )
        ];
      };
    };
}
```

### Using the overlay

Alternatively, if you don't want to use our `packages` output, you can add our overlay to your nixpkgs instance.
This will ensure PineconeMC is built with your system's packages.

> [!WARNING]
> Depending on what revision of nixpkgs your system uses, this may result in binaries that differ from the above `packages` output
> If this is the case, you will not be able to use the binary cache

Example:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

    pineconemc = {
      url = "github:Impqxr/LauncherNix";

      # Optional: Override the nixpkgs input of pineconemc to use the same revision as the rest of your flake
      # Note that this may break the reproducibility mentioned above, and you might not be able to access the binary cache
      #
      # inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs =
    { nixpkgs, pineconemc, ... }:
    {
      nixosConfigurations.foo = nixpkgs.lib.nixosSystem {
        modules = [
          ./configuration.nix

          (
            { pkgs, ... }:
            {
              nixpkgs.overlays = [ pineconemc.overlays.default ];

              environment.systemPackages = [ pkgs.pineconemc ];
            }
          )
        ];
      };
    };
}
```

### Installing the package ad-hoc (`nix shell`, `nix run`, etc.)

You can simply call the default package of this flake.

Example:

```shell
nix run github:Impqxr/LauncherNix

nix shell github:Impqxr/LauncherNix

nix profile install github:Impqxr/LauncherNix
```

## Installing a development release (without flakes)

We use [Cachix](https://cachix.org/) to cache PineconeMC builds.
If you want to avoid rebuilds you may add Cachix to your substitutors.

Example (NixOS):

```nix
{
  nix.settings = {
    trusted-substituters = [ "https://pineconemc-unofficial.cachix.org" ];

    trusted-public-keys = [
      "pineconemc-unofficial.cachix.org-1:aFVxmb1DKgTEdeaCUATWQXz73nm6oPI2r+Vd+wfgR/0="
    ];
  };
}
```

### Installing the package directly (`fetchTarball`)

We use flake-compat to allow using this Flake on a system that doesn't use flakes.

Example:

```nix
{ pkgs, ... }:
{
  environment.systemPackages = [
    (import (
      builtins.fetchTarball "https://github.com/Impqxr/LauncherNix/archive/develop.tar.gz"
    )).packages.${pkgs.system}.pineconemc
  ];
}
```

### Using the overlay (`fetchTarball`)

Alternatively, if you don't want to use our `packages` output, you can add our overlay to your instance of nixpkgs.
This results in PineconeMC using your system's libraries

Example:

```nix
{ pkgs, ... }:
{
  nixpkgs.overlays = [
    (import (
      builtins.fetchTarball "https://github.com/Impqxr/LauncherNix/archive/develop.tar.gz"
    )).overlays.default
  ];

  environment.systemPackages = [ pkgs.pineconemc ];
}
```

### Installing the package ad-hoc (`nix-env`)

You can add this repository as a channel and install its packages that way.

Example:

```shell
nix-channel --add https://github.com/Impqxr/LauncherNix/archive/develop.tar.gz pineconemc

nix-channel --update pineconemc

nix-env -iA pineconemc.pineconemc
```

## Package variants

Both Nixpkgs and this repository offer the following packages:

- `pineconemc` - The preferred build, wrapped with everything necessary to run the launcher and Minecraft
- `pineconemc-unwrapped` - A minimal build that allows for advanced customization of the launcher's runtime environment

### Customizing wrapped packages

The wrapped package (`pineconemc`) offers some build parameters to further customize the launcher's environment.

The following parameters can be overridden:

- `additionalLibs` (default: `[ ]`) Additional libraries that will be added to `LD_LIBRARY_PATH`
- `additionalPrograms` (default: `[ ]`) Additional libraries that will be added to `PATH`
- `controllerSupport` (default: `isLinux`) Turn on/off support for controllers on Linux (macOS will always have this)
- `gamemodeSupport` (default: `isLinux`) Turn on/off support for [Feral GameMode](https://github.com/FeralInteractive/gamemode) on Linux
- `jdks` (default: `[ jdk25 jdk21 jdk17 jdk8 ]`) Java runtimes added to `PINECONEMC_JAVA_PATHS` variable
- `msaClientID` (default: `null`, requires full rebuild!) Client ID used for Microsoft Authentication
- `textToSpeechSupport` (default: `isLinux`) Turn on/off support for text-to-speech on Linux (macOS will always have this)
