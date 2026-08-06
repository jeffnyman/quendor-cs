# Quendor

_A Specification-Accurate Z-Machine Implementation_

## Getting the source

```sh
git clone https://github.com/jeffnyman/quendor-cs.git quendor
cd quendor
```

## Building

See "Prerequisites" below.

From the repository root:

```sh
dotnet build
```

That picks up `Quendor.slnx` and builds every project. Warnings are
treated as errors, so a build that emits nothing is a build that is clean.

## Running

```sh
dotnet run --project src/Quendor.Cli
```

The executable is named `quendor`, not `Quendor.Cli`. To produce a
standalone build:

```sh
dotnet publish src/Quendor.Cli -c Release -o ./publish
./publish/quendor          # .\publish\quendor.exe on Windows
```

## Testing

```sh
dotnet test
```

To run a subset:

```sh
dotnet test --filter "FullyQualifiedName~Header"
```

To collect coverage (writes a Cobertura report under
`tests/*/TestResults/`):

```sh
dotnet test --collect:"XPlat Code Coverage"
```

Tests use [xUnit v3](https://xunit.net/).

## Contributing

Formatting and naming are governed by `.editorconfig`. Before opening a pull
request:

```sh
dotnet format
dotnet build
dotnet test
```

CI runs the same checks on Linux, Windows, and macOS, plus
`dotnet format --verify-no-changes` and a line-ending check.

Three conventions worth knowing about up front:

- **Warnings are errors.** `TreatWarningsAsErrors` is on and the analysis
  level is `latest-recommended`.
- **Culture-sensitive APIs will fail the build.** Calls like `ToLower()`,
  `int.ToString()`, and `string.StartsWith(string)` trip CA1304/CA1305/CA1310
  because their behavior depends on the machine's locale. This is deliberate:
  a Z-Machine's text handling is defined by the specification, not by the
  user's regional settings. Under a Turkish locale, for instance,
  `"INVENTORY".ToLower()` yields `"ınventory"` with a dotless i, which would
  silently break dictionary lookup. Use the ZSCII rules from the spec, or an
  explicitly invariant/ordinal overload.
- **Commit messages follow
  [Conventional Commits](https://www.conventionalcommits.org/).** A
  `commit-msg` hook, installed automatically the first time you build,
  rejects messages that do not fit `type(optional-scope): description`,
  and CI applies the same check to pull request titles, which become the
  commit on `main` when squash-merged. The history is machine-readable on
  purpose: version bumps and the changelog are derived from it.

## Prerequisites

You will need the .NET SDK. The version is pinned in `global.json`, so you
need **10.0.302 or newer** in the 10.0.x band — an older SDK will refuse to
build rather than silently doing the wrong thing.

Verify what you have with `dotnet --list-sdks`.

### Windows

```powershell
winget install -e --id Microsoft.VisualStudio.BuildTools --source winget
winget install -e --id Microsoft.DotNet.SDK.10 --source winget
```

- `-e` enforces an exact match for the package ID.
- `--id` pinpoints the explicit unique identifier for .NET 10.
- `--source winget` ensures Windows Package Manager pulls the manifest directly from the official WinGet repository instead of third-party store listing.

### macOS

```sh
brew install --cask dotnet-sdk
```

### Ubuntu / Debian-based systems

```sh
sudo apt update
sudo apt install -y dotnet-sdk-10.0
```

### Fedora / RHEL-based systems

```sh
sudo dnf install -y dotnet-sdk-10.0
```

### Arch Linux

```sh
sudo pacman -S dotnet-sdk
```

### Any platform

Distribution repositories often lag behind. If the package above is
unavailable or too old, use the official installer script, which needs no
root and drops the SDK in `~/.dotnet`:

```sh
curl -sSL https://dot.net/v1/dotnet-install.sh | bash -s -- --channel 10.0
```

That is everything needed to build, run, and test.

## Reference Material (optional)

An `entharion` submodule holds the specifications and story files this
project is developed against. These are not required as part of building
and deploying Quendor, but they help during development.

If you want this reference material and if this is your first time checking
out the repo, run this command:

```sh
git submodule update --init --recursive
```

### Building the Reference Tools

Entharion includes three buildable C tools, each a nested submodule:

- `frotz` — the reference Z-Machine interpreter; its "dumb" build
  (`dfrotz`) runs in a plain terminal with no display dependencies.
- `ztools` — inspection utilities such as `infodump` (header, objects,
  dictionary) and `txd` (disassembler).
- `reform6` — an Inform 6 based compiler for producing story files.

Building them is optional. They are useful for comparing Quendor's
behavior against known-good implementations. All three need only a C
compiler, `make`, and a Unix-like environment.

### Prerequisites

**Windows.** The tools assume a Unix environment, so use WSL. From an
elevated PowerShell (rebooting if prompted, then creating a Unix user
when the Ubuntu shell first opens):

```powershell
wsl --install
```

Then, inside the Ubuntu shell, install the toolchain:

```sh
sudo apt update
sudo apt install build-essential groff
```

(`groff` is only needed to format the ztools man pages.)

Your Windows drives are visible in WSL under `/mnt`, so a checkout at
`F:\Projects\quendor-rs` is reachable at `/mnt/f/Projects/quendor-rs`.

**macOS.** Install the command line developer tools:

```sh
xcode-select --install
```

**Linux.** Install a compiler toolchain, e.g. on Debian/Ubuntu:

```sh
sudo apt update
sudo apt install build-essential groff
```

### Compiling

From the repository root (in WSL, macOS Terminal, or a Linux shell):

```sh
make -C entharion/ztools
make -C entharion/reform6
make -C entharion/frotz dumb
```

The binaries land in each tool's own directory, and each of those
repositories already ignores its build artifacts, so nothing shows up
as untracked in Git.

### Running

From a Unix shell:

```sh
./entharion/frotz/dfrotz entharion/zcode-infocom/ballyhoo-r97-s851218.z3
./entharion/ztools/infodump -i entharion/zcode-infocom/amfv-r77-s850814.z4
```

On Windows the binaries are Linux executables, but they can be invoked
directly from PowerShell by prefixing `wsl`:

```powershell
wsl ./entharion/frotz/dfrotz entharion/zcode-infocom/ballyhoo-r97-s851218.z3
```

## License

MIT. See [LICENSE](LICENSE).
