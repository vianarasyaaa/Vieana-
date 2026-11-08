# Directory Rules

- `Vieana/`: production C++26 source, internal shader source and production resources only.
- `Testing/`: all test code, fixtures, generated images, golden results, benchmark output, profiling captures, logs, crash dumps and temporary test artifacts.
- `Build/`: all compiler outputs, intermediate objects, generated headers, generated shaders/SPIR-V, caches, APKs, symbols and packaging outputs.
- `Documentation/`: architecture, specifications, research, APIs and engineering decisions.

Never add `tests/`, `benchmarks/`, `fixtures/`, `coverage/`, `logs/` or generated outputs beneath `Vieana/`. Tests may include Vieana headers and link internal builds; their data and output remain in `Testing/`. Build output is external to source and reproducible.
