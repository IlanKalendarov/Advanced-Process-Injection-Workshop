# AGENTS.md

## Cursor Cloud specific instructions

### Project Overview

This is the **CyberWarFare Labs Advanced Process Injection Workshop** — a collection of Windows-native C++ projects demonstrating various process injection techniques. Each subdirectory contains a Visual Studio 2022 solution (`.sln`) and project files (`.vcxproj`).

### Platform Constraint

This codebase is **Windows-only**. All projects use Win32/NT APIs (`Windows.h`, `ntdll.dll`, `TlHelp32.h`, `Psapi.h`, etc.) and are designed to be built with MSVC v143 (Visual Studio 2022) and run on Windows 10+ with elevated privileges.

### Cross-compilation on Linux (Cloud VM)

On the Linux cloud VM, projects can be **cross-compiled** using MinGW-w64 (`x86_64-w64-mingw32-g++` for x64, `i686-w64-mingw32-g++` for x86). The resulting `.exe`/`.dll` files are valid Windows PE binaries but **cannot be executed on Linux**.

Key cross-compilation notes:

- **Case-sensitive headers**: Linux is case-sensitive; the source uses `Windows.h`, `TlHelp32.h`, `Psapi.h` but MinGW provides lowercase variants. A compat-include directory (`build/compat-includes/`) with wrapper headers is needed. Create it with:
  ```
  mkdir -p build/compat-includes
  echo '#include <windows.h>' > build/compat-includes/Windows.h
  echo '#include <tlhelp32.h>' > build/compat-includes/TlHelp32.h
  echo '#include <psapi.h>' > build/compat-includes/Psapi.h
  ```
- **UNICODE**: Most projects need `-DUNICODE -D_UNICODE` for correct `PROCESSENTRY32` / wide-char API usage.
- **Linker flags**: Projects using NT APIs need `-lntdll`; Module Stomping needs `-lpsapi`; DLL project needs `-shared`.
- **Process Hollowing** is x86-only (`i686-w64-mingw32-g++`) due to 32-bit pointer arithmetic.

### Build commands (cross-compilation)

All commands are run from the workspace root (`/workspace`):

```bash
# HelloWorld (x64 and x86)
x86_64-w64-mingw32-g++ -I build/compat-includes -o build/HelloWorld.exe HelloWorld/HelloWorld/Hello.cpp -mwindows -static
i686-w64-mingw32-g++ -I build/compat-includes -o build/HelloWorld_x86.exe HelloWorld/HelloWorld/Hello.cpp -mwindows -static

# APC Injection (x64)
x86_64-w64-mingw32-g++ -I build/compat-includes -I CWLAPCInjection/CWLAPCInjection -DUNICODE -D_UNICODE -o build/CWLAPCInjection.exe CWLAPCInjection/CWLAPCInjection/CWLImplant.cpp -mwindows -static -lntdll

# CWLDLL (shared library)
x86_64-w64-mingw32-g++ -I build/compat-includes -DUNICODE -D_UNICODE -shared -o build/CWLDLL.dll CWLDLL/CWLDLL/dllmain.cpp -mwindows -static

# Module Stomping (x64)
x86_64-w64-mingw32-g++ -I build/compat-includes -I CWLModuleStomping/CWLModuleStomping -DUNICODE -D_UNICODE -o build/CWLModuleStomping.exe CWLModuleStomping/CWLModuleStomping/CWLImplant.cpp -mwindows -static -lpsapi

# Process Hollowing (x86 only)
i686-w64-mingw32-g++ -I build/compat-includes -I ProcessHollowing/ProcessHollowing -o build/ProcessHollowing_x86.exe ProcessHollowing/ProcessHollowing/CWLImplant.cpp -mwindows -static -lntdll

# Process Doppelganging (x64)
x86_64-w64-mingw32-g++ -I build/compat-includes -I CWLProcessDoppelganging/CWLProcessDoppelganging -DUNICODE -D_UNICODE -o build/CWLProcessDoppelganging.exe CWLProcessDoppelganging/CWLProcessDoppelganging/CWLImplant.cpp -mwindows -static -lntdll

# Herpaderping (x64)
x86_64-w64-mingw32-g++ -I build/compat-includes -I CWLHerpaderping/CWLHerpaderping -DUNICODE -D_UNICODE -o build/CWLHerpaderping.exe CWLHerpaderping/CWLHerpaderping/CWLImplant.cpp -mwindows -static -lntdll

# Process Ghosting (x64)
x86_64-w64-mingw32-g++ -I build/compat-includes -I CWLProcessGhosting/CWLProcessGhosting -DUNICODE -D_UNICODE -o build/CWLProcessGhosting.exe CWLProcessGhosting/CWLProcessGhosting/CWLImplant.cpp -mwindows -static -lntdll

# Transacted Hollowing (x64)
x86_64-w64-mingw32-g++ -I build/compat-includes -I CWLTransactHollowing/CWLTransactHollowing -DUNICODE -D_UNICODE -o build/CWLTransactHollowing.exe CWLTransactHollowing/CWLTransactHollowing/CWLImplant.cpp -mwindows -static -lntdll
```

### No lint, test, or CI infrastructure

This repository has no automated tests, no lint configuration, and no CI/CD pipeline. Validation is done by building the projects and verifying the output files are valid Windows PE executables (`file *.exe`).

### Execution requirements (Windows only)

To actually run these tools, you need:
- Windows 10+ with administrator/elevated privileges
- Payloads placed in `C:\temp\` (see `payloads/` directory in repo)
- Target processes running (e.g., `notepad.exe`)
- Refer to `README.md` and the workshop PDF for detailed setup instructions.
