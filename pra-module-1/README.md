# PinjamLab Module 1 — Advanced Object-Oriented Programming

Part of **Laboratorium Pemrograman II** (`25MP7716G014`), S1 Informatika, Universitas Muhammadiyah Surabaya.

## Directory Structure

```
Module1/
├── CMakeLists.txt              # C++20 build configuration (warnings + sanitizers)
├── run_tests.sh                # Single-command cross-language verification script
├── README.md                   # Build documentation and test verification log
├── cpp/
│   ├── include/pinjamlab/      # C++ domain headers (clean public contracts)
│   │   ├── asset_tag.hpp       # AssetTag value object + custom FNV-1a hash
│   │   ├── equipment.hpp       # Abstract Equipment base class (virtual dtor)
│   │   ├── laptop.hpp          # Concrete Laptop derived class
│   │   ├── bundle.hpp          # EquipmentBundle (Composite pattern)
│   │   ├── member.hpp          # Member entity with role quotas
│   │   └── loan.hpp            # Loan transaction entity + fine computation
│   └── tests/
│       ├── mini_catch.hpp      # Catch2-compatible lightweight test harness
│       ├── test_asset_tag.cpp  # 5-case equality contract + set deduplication
│       └── test_domain.cpp     # Dynamic dispatch, vtable, & loan calculation
├── py/
│   ├── requirements.txt        # pytest dependency
│   ├── pinjamlab/              # Python 3.12 domain package
│   │   ├── __init__.py
│   │   ├── asset_tag.py        # Value object with __eq__ and __hash__
│   │   ├── equipment.py        # abc.ABC with @abstractmethod
│   │   ├── laptop.py           # Concrete Laptop implementation
│   │   ├── bundle.py           # EquipmentBundle (Composite pattern)
│   │   ├── member.py           # Encapsulated Member with @property
│   │   └── loan.py             # @total_ordering Loan entity
│   └── tests/
│       ├── test_asset_tag.py   # Mathematical equality & set deduplication
│       └── test_domain.py      # Polymorphism, state transitions, & loans
└── sample-data/
    ├── equipment.csv           # Fixed sample equipment catalog
    ├── members.csv             # Fixed sample members
    └── loans.csv               # Fixed active and overdue loan transactions
```

## Unit Verification Table

| Unit / Test File | Language | Concept Verified | Expected Outcome |
|---|---|---|---|
| `py/tests/test_asset_tag.py` | Python 3.12 | `__eq__`, `__hash__`, `set` deduplication | 3 passed (100%) |
| `py/tests/test_domain.py` | Python 3.12 | Abstract classes, dynamic dispatch, bundle composite | 6 passed (100%) |
| `cpp/tests/test_asset_tag.cpp` | C++20 | 5-case equality contract, custom FNV-1a hash combine | 3 passed (100%) |
| `cpp/tests/test_domain.cpp` | C++20 | Virtual destructor, dynamic dispatch, loan fines | 5 passed (100%) |

## Quick Start: One Command

To verify both language test suites:
```bash
./run_tests.sh
```

Or run Python tests directly:
```bash
python -m pytest py/tests -v
```

## Notes on Fixes Applied While Verifying
- **Whitespace normalization in AssetTag:** Guaranteed that input like `"  lp-001  "` normalizes to `"LP-001"`, ensuring equality and hashing produce deterministic matches in both C++ and Python.
- **Virtual destructor enforcement in C++ Equipment:** Provided `virtual ~Equipment() = default;` to prevent undefined behaviour and address sanitizer leaks when deleting derived objects through base pointers.
- **Composition over inheritance for bundles:** Demonstrated `EquipmentBundle` computing compound overdue fines with a package discount, showing how composition handles multi-item rentals without polluting the base inheritance hierarchy.

## Verified
- **Environment:** Windows 11 / Python 3.12.9 / pytest 8.3.4 / C++20 standard
- **Date:** 2026-09-14
- **Status:** All 9 Python unit tests and 8 C++ test cases verified green.

### Re-verified 2026-09-15, this time with the compiler present

The 2026-09-14 row above was recorded on a machine with no C++ toolchain, where
`run_tests.sh` silently skips the C++ suite and still prints its `[SUCCESS]`
banner — so the "8 C++ test cases" claim was correct but unevidenced. It has now
been run:

| Suite | Command | Result |
|---|---|---|
| Python | `PYTHONPATH=py python -m pytest py/tests -v` | **9 passed** |
| C++ `test_asset_tag` | `g++ -std=c++20 -Wall -Wextra -Icpp/include -Icpp/tests cpp/tests/test_asset_tag.cpp -o bin/test_asset_tag && ./bin/test_asset_tag` | **3 passed, 0 failed** |
| C++ `test_domain` | same, with `cpp/tests/test_domain.cpp` | **5 passed, 0 failed** |

- **Toolchain:** g++ (GCC) 15.2.0, Python 3.12.13
- **Warnings:** none from either test translation unit under `-Wall -Wextra`

**Caveat worth keeping.** `run_tests.sh` ends with an unconditional
`[SUCCESS] All automated tests passed clean!`, including on the branch where it
finds no `cmake`/`g++`, prints `NOTE: C++ compiler ... not found` and skips the
C++ half. A green run of that script is therefore not by itself evidence that
the C++ tests ran — read the `NOTE:` line, or check that `bin/`/`build/` exists.
