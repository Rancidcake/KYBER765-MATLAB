Kyber-MATLAB
============

This project provides MATLAB and Simulink bindings for the Kyber768 post-quantum KEM (Key Encapsulation Mechanism).  
It wraps the reference C implementation of Kyber into MATLAB using MEX files, and includes a simple smoke test and example GUI hooks.

Contents
--------
kyber-matlab/
  external/kyber     → Reference Kyber C sources  
  src/mex/           → C++ MEX wrapper files  
  matlab/            → MATLAB entrypoints, smoke tests, build scripts  
  simulink/          → Example Simulink blocks  
  Readme.txt         → This file  

Requirements
------------
- Windows 10/11 (x64) or macOS (Sequoia or later)
- MATLAB R2025a (or later) with:
  - MATLAB Coder
  - Simulink (optional, for Simulink integration)
- C++ Compiler:
  - Windows: Microsoft Visual Studio (Community Edition is fine, install Desktop Development with C++ workload)
  - macOS: Xcode Command Line Tools
- MATLAB must detect your C++ compiler (`mex -setup C++`)

Build Instructions
------------------
**Windows:**
1. Open MATLAB.
2. Set up the compiler (one-time):  
   `mex -setup C++`  
   Select Microsoft Visual C++.
3. Move into the MATLAB folder:  
   `cd('path\\to\\kyber-matlab\\matlab')`
4. Run the build script:  
   `build_mex`

**macOS:**
1. Open MATLAB.
2. Set up the compiler (one-time):  
   `mex -setup C++`  
   Select Xcode.
3. Move into the MATLAB folder:  
   `cd('path/to/kyber-matlab/matlab')`
4. Run the build script:  
   `build_mex`

This will compile three MEX files:
- kyber768_keygen_mex.mexw64 / .mexmaci64
- kyber768_encaps_mex.mexw64 / .mexmaci64
- kyber768_decaps_mex.mexw64 / .mexmaci64

> ⚠️ Note: On Windows, the extension is `.mexw64`. On macOS, it is `.mexmaci64`.

Smoke Test
----------
After building, run:
```
demo_smoke_test
```
Expected output:
```
Keygen...
Encaps...
Decaps...
OK! Shared secret match. ss = <hex string>
```

Using in MATLAB
---------------
High-level MATLAB wrappers:
- `kyber768_keygen.m` → generates key pair
- `kyber768_encaps.m` → encapsulates shared secret
- `kyber768_decaps.m` → decapsulates shared secret

Example:
```matlab
[pk, sk] = kyber768_keygen();
[ct, ss_enc] = kyber768_encaps(pk);
ss_dec = kyber768_decaps(ct, sk);

isequal(ss_enc, ss_dec)   % should be true
```

Simulink Integration
--------------------
Use a MATLAB Function block in Simulink.  
Inside the block, call the wrappers above.

Example function body:
```matlab
function out = demoKyber(~)
  [pk, sk] = kyber768_keygen();
  [ct, ss1] = kyber768_encaps(pk);
  ss2 = kyber768_decaps(ct, sk);
  out = isequal(ss1, ss2);
end
```
This outputs a boolean true if the KEM round-trip succeeds.

Known Issues / Notes
--------------------
- Do **not** build files in `external/kyber/nistkat` or `external/kyber/test` (these are for NIST KAT harnesses and require OpenSSL).
- Warnings like “treating .c as C++” are harmless.
- For GUI demos, see `mini_gui.m` (optional).

Quick Troubleshooting
--------------------
- **Error: cl.exe not found**  
  → Install Visual Studio with Desktop C++ workload and restart MATLAB.
- **Error: Undefined symbol crypto_kem_\***  
  → Ensure you compiled with all core Kyber .c files (see `build_mex.m`).
- **Error: _mexFunction missing**  
  → Add `-R2018a` in the build flags (MATLAB R2025a uses the modern C++ MEX API).
Kyber-MATLAB on Windows (R2025a)
================================

0) What you’ll build
--------------------
Three MEX gateways wrapping the Kyber768 C reference:
- kyber768_keygen_mex.mexw64
- kyber768_encaps_mex.mexw64
- kyber768_decaps_mex.mexw64

You’ll then call them from the MATLAB wrappers in `matlab/` and run a smoke test.

1) Prerequisites
----------------
- Windows 10/11 x64
- MATLAB R2025a (installed)
- Microsoft Visual Studio (MSVC) toolchain  
  Easiest: Visual Studio 2022 (Community) → install the “Desktop development with C++” workload.  
  Minimal alternative: Build Tools for Visual Studio 2022 with MSVC, Windows SDK, and CMake components.
- (Optional) Simulink if you want the Simulink example.
- No OpenSSL is required (we do not compile the nistkat/ and test/ harnesses).

2) Get the source
-----------------
Example layout:
```
C:\work\kyber-matlab\
  external\kyber\
  src\mex\
  matlab\
  simulink\
```
Clone or copy the repo so these folders exist with files inside.

3) Set MSVC as MATLAB’s compiler
--------------------------------
Open MATLAB (R2025a) → set C++ compiler once:
```
mex -setup C++
```
Choose Microsoft Visual C++ (e.g., “Microsoft Visual C++ 2022 (C++)”).

Verify:
```
mex -setup -v C++
```
You should see `Selected C++ compiler: Microsoft Visual C++ ...` in the output.

4) Build script (Windows)
-------------------------
Go to the MATLAB folder:
```
cd('C:\work\kyber-matlab\matlab')
```
Create or overwrite `build_mex.m` with the content below.  
This builds only the core Kyber sources (no OpenSSL / nistkat / tests).

```matlab
function build_mex(verbose)
% BUILD_MEX  Build Kyber768 MEX gateways on Windows (R2025a).
% Usage:
%   build_mex          % normal build
%   build_mex(true)    % verbose MEX output

if nargin < 1, verbose = false; end

% Paths (Windows-friendly)
root      = fileparts(fileparts(mfilename('fullpath'))); % ..\ from matlab\
incKyber  = fullfile(root, 'external', 'kyber');
srcKyber  = fullfile(root, 'external', 'kyber');         % core C files
srcMex    = fullfile(root, 'src', 'mex');

% Core Kyber C sources (no nistkat/, no test/)
coreC = { ...
  fullfile(srcKyber,'cbd.c'), ...
  fullfile(srcKyber,'fips202.c'), ...
  fullfile(srcKyber,'indcpa.c'), ...
  fullfile(srcKyber,'kem.c'), ...
  fullfile(srcKyber,'ntt.c'), ...
  fullfile(srcKyber,'poly.c'), ...
  fullfile(srcKyber,'polyvec.c'), ...
  fullfile(srcKyber,'randombytes.c'), ...
  fullfile(srcKyber,'reduce.c'), ...
  fullfile(srcKyber,'symmetric-shake.c'), ...
  fullfile(srcKyber,'verify.c') ...
};

% Common mex flags
mexCommon = { ...
  '-R2018a', ...                     % use modern C++ MEX API
  ['-I' incKyber] ...
};

if verbose
  mexCommon = [{'-v'}, mexCommon];   % verbose compile
end

targets = { ...
  {'kyber768_keygen_mex',  fullfile(srcMex,'kyber768_keygen_mex.cpp')}, ...
  {'kyber768_encaps_mex',  fullfile(srcMex,'kyber768_encaps_mex.cpp')}, ...
  {'kyber768_decaps_mex',  fullfile(srcMex,'kyber768_decaps_mex.cpp')}
};

for k = 1:numel(targets)
  name = targets{k}{1};
  cpp  = targets{k}{2};
  fprintf('Building %s...\n', name);
  mex(mexCommon{:}, cpp, coreC{:});
end

fprintf('Done. Produced .mexw64 files next to the current folder.\n');
end
```

Notes:
- `-R2018a` is included to ensure the correct C++ MEX API entry points are linked (prevents “_mexFunction undefined” LNK errors).
- We intentionally exclude `external\kyber\nistkat\*.c` and `external\kyber\test\*.c` to avoid OpenSSL and test harness dependencies.

Run it:
```
build_mex        % or build_mex(true) for verbose
```
You should see three `.mexw64` files produced in the current folder (MATLAB puts them where you run mex; if needed, `pwd` to confirm location).

5) Sanity check: the MEX files are visible
------------------------------------------
```
which kyber768_keygen_mex -all
which kyber768_encaps_mex -all
which kyber768_decaps_mex -all
```
Each should resolve to a `.mexw64` path. If not, make sure MATLAB’s current folder is where the MEX files landed, or add the folder to the path:
```
addpath(pwd)           % if the .mexw64 live in the current folder
% or:
addpath('C:\work\kyber-matlab\matlab')
savepath               % optional
```

6) Run the smoke test
---------------------
From `C:\work\kyber-matlab\matlab`:
```
demo_smoke_test
```
Expected:
```
Keygen...
Encaps...
Decaps...
OK! Shared secret match.
```
Or call the wrappers manually:
```matlab
[pk, sk]      = kyber768_keygen();
[ct, ss_enc]  = kyber768_encaps(pk);
ss_dec        = kyber768_decaps(ct, sk);
isequal(ss_enc, ss_dec)  % should be true
```

7) (Optional) Simulink quick demo
---------------------------------
Open or create a model.  
Add a MATLAB Function block.  
Double-click it → paste:
```matlab
function ok = demoKyber()
%#codegen
  [pk, sk]   = kyber768_keygen();
  [ct, ss1]  = kyber768_encaps(pk);
  ss2        = kyber768_decaps(ct, sk);
  ok         = isequal(ss1, ss2);
end
```
Simulate—the output `ok` should be true.

8) Common Windows issues & fixes
-------------------------------
- **LNK2019: unresolved external symbol _mexFunction**  
  You compiled a C++ MEX that uses the modern API but linked against the legacy entry.  
  Fix: ensure `-R2018a` is in your mex call (already in build_mex.m).  
  Also confirm you’re building the *_mex.cpp files from `src\mex\`, not the plain C files directly.

- **cl.exe not found / MATLAB picks MinGW**  
  Re-run `mex -setup C++` and pick Microsoft Visual C++.  
  Ensure Visual Studio (or Build Tools) is installed with Desktop development with C++.

- **MEX builds but MATLAB can’t find the file**  
  You compiled in a different folder. Run `which kyber768_keygen_mex -all`.  
  Use `addpath('<folder-with-mexw64>')`.

- **Rebuild after changing sources**  
  ```
  clear mex; rehash toolboxcache
  build_mex(true)
  ```

- **Do not include OpenSSL**  
  If you see errors about `<openssl/...>`, you accidentally pulled in files from `external\kyber\nistkat\` or `external\kyber\test\`. Only build the core list in `build_mex.m`.

9) Clean
--------
Delete built MEX files (from the folder they were written to):
```
delete('kyber768_*_mex.mexw64')
clear mex
rehash toolboxcache
```

10) Hand-off checklist
----------------------
- Visual Studio C++ installed, `mex -setup C++` selects MSVC
- `build_mex.m` exactly as above (Windows paths)
- Three `.mexw64` files present and discoverable by `which`
- `demo_smoke_test` passes
