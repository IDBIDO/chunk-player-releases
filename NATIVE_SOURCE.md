# Native corresponding-source delivery

This source delivery accompanies the unchanged LGPL libmpv runtime used by
Chunk Player Windows beta releases. It is separate from the private player
application source. The public release channel is
[Chunk Player releases](https://github.com/IDBIDO/chunk-player-releases/releases/tag/v0.2.0-beta.1).
Download `chunk-player-libmpv-source-e71a429.tar.gz` and verify its SHA-256
against that release's `SHA256SUMS.txt` before extracting it.

## Exact identities and scope

- Native build recipe commit: `e71a429758e3cd03fb36e5f7903efaf0a8a1ab1a`.
- Source lock SHA-256: `9ec1f1d627db876103476c6be59b65dd74481ed56eddbe098fc92d9721f2bbcd`.
- `libmpv-2.dll` SHA-256: `7777cab2015465b62eb74d23df4b0554c1a426d71ea4311fd4f4927824800bba`.
- Baseline recipe: upstream `media-kit/libmpv-win32-video-cmake`, commit
  `8ddbe5472465950b87853789f7173f2eedc5586a`.

`SOURCE_INPUTS.json` accounts for all 530 locked inputs. It contains 67
self-contained upstream Git bundles and 458 unchanged source archives. The
upstream baseline has its own `baseline.bundle`. Source archives include the
linked library sources and their dependencies, plus conservatively retained
build inputs; presence in this source delivery does not mean an optional
codec, encoder, or library is enabled in the shipped runtime.

Five binary build prerequisites are identified by original URL, byte size and
SHA-256 instead of being redistributed: the four locked Rust compiler/standard
library archives and Microsoft's `Windows.winmd`. They can be obtained by the
restore command below. The pinned Linux build-tool container is another
external standard-tool prerequisite. These are not omitted libmpv/FFmpeg
sources. The player, dictionary, private media, user database, speech models,
secrets, local paths, raw logs, cache directories and private Git history are
not part of this delivery.

`native-build/` contains the exact reviewed container scripts, source locks,
policy/configuration, patches, build scripts, licenses/notices and Makefile.
Original build glue is supplied under `LICENSE-native-build.txt`; upstream
code and patches retain the licenses that apply to them. Every included file
except the self-describing `FILE_INVENTORY.json` has a relative path, byte size
and SHA-256 in that inventory. Git bundles contain upstream objects needed
for the pinned source commits, not the distributor's Git configuration,
reflogs, credentials or private application history.

## Restore and rebuild on Linux

Use Linux x86-64, Python 3.12 or later, Git, Podman and Skopeo. Allocate at least
30 GB of free space for restored source inputs, the toolchain and a build;
actual needs depend on build parallelism and retained output. The first two
steps obtain the pinned standard build tools. The compilation container is
then run with networking disabled and image pulling forbidden.

1. From the extracted `chunk-player-libmpv-source` directory, restore into a
   new sibling directory (it must not already exist):

   ```sh
   python3 restore.py --destination ../native-rebuild
   cd ../native-rebuild
   ```

   This verifies package file hashes, restores every locked Git commit,
   checks every archive hash and generates the materialization manifest.
   It downloads only the five explicit prerequisites described above.
   `--external-download-cache` can instead point to predownloaded, hash-verified
   originals using the native build cache filename convention.

2. Obtain and load the locked toolchain image without resolving `latest`:

   ```sh
   skopeo copy docker://ghcr.io/shinchiro/archlinux@sha256:8a379b2f2689e1790ad3e504a6274ffde32b88068cfad0e5d94b13782d1c5fca oci-archive:.cache/toolchain-image.oci
   python3 container/resolve-image.py verify-archive --lock locks/toolchain-image.lock.json --archive .cache/toolchain-image.oci
   podman load --input .cache/toolchain-image.oci
   ```

3. Build the product recipe into a new output directory:

   ```sh
   python3 scripts/build.py --recipe product --output out/recipient-build
   ```

   Build options are recorded in `policy/release-policy.json`; the script
   applies `patches/`, locks the target and Rust inputs, renders the upstream
   recipe, and captures effective FFmpeg options and final link inventory.
   The reviewed original build disabled GPL/nonfree FFmpeg features, encoders,
   DVD inputs and LZO; the effective configuration is the evidence to inspect,
   rather than inferring enabled features from all bundled source inputs.
   A successful command is engineering evidence of a build; it is not a new
   distribution approval for a recipient's modified artifact.

4. For a local modification, edit a restored source tree and use a recipient
   copy of the recipe. Its immutable-input checks deliberately reject changes
   until its source lock/materialization checks are updated to describe the
   recipient's new inputs. Keep a copy of the original source and record your
   changes. The scripts are supplied so those checks can be understood and
   modified, not to prevent recipients from rebuilding modified libraries.

## Library replacement and beta2 packaging

The player dynamically loads `libmpv-2.dll`. Close the player, retain the
original DLL as a backup, and replace it in the installation directory with
an ABI-compatible Windows x64 library for local testing. Rebuilding a library
with statically linked dependencies means relinking those dependencies into
that DLL; no private player source is necessary to replace the shared library.
This permission includes debugging modifications to the LGPL components.
Restore the original DLL if the modified library is incompatible.

`repair/` contains the separate `0.1.0-beta.2` SDK repack recipe and manifest.
That repair adds the two missing public ANGLE header dependencies from their
already pinned upstream commit. It leaves all four runtime DLLs byte-identical.
Follow `repair/README.md` for the repack command. The original beta1 native ZIP
is an explicit repack input; it is not required to rebuild libmpv from source.
A rebuilt, modified runtime should receive its own package identity instead
of being described as the byte-identical beta2 runtime.

## License delivery and requests

Full license texts and notices are retained under `native-build/third_party/`,
including the LGPL and GPL texts for covered libraries. The installer also
carries its applicable full native license files. Upstream runtime components
obtained as permissively licensed binaries retain the supplier's notices and
source identities; this archive does not claim to reproduce Microsoft's
proprietary compiler DLL or rebuild the separate prebuilt ANGLE runtime.

Providing corresponding source and allowing compatible shared-library
replacement are part of the requirements described in the official
[LGPL v3](https://www.gnu.org/licenses/lgpl-3.0.html) and
[GPL v3](https://www.gnu.org/licenses/gpl-3.0.html). The shipped source and
license files provide the concrete inputs; a manifest of URLs alone would
not deliver those source inputs.

If an archive or prerequisite becomes unavailable, or a corresponding-source
input is missing, request assistance through the public
[release repository issues](https://github.com/IDBIDO/chunk-player-releases/issues/new),
quoting the native commit and source-lock SHA-256 above. Do not attach private
media, subtitle text, account credentials, or a personal database.
