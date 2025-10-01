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

Final Word
----------
This setup has been validated on macOS (Sequoia) + MATLAB R2025a and is portable to Windows 11 + MATLAB R2025a with MSVC. Once built, the API behaves the same across platforms.
