Here is the complete setup to compile the modernized Rust version into a Python library using Maturin.

1. File Structure

Set up your project directory structure exactly like this:
text
my_project/
├── Cargo.toml
├── pyproject.toml
└── src/
    └── lib.rs
Use code with caution.

2. Configuration Files


pyproject.toml

This file configures Python's build system to use Maturin as the backend.
toml
[build-system]
requires = ["maturin>=1.0,<2.0"]
build-backend = "maturin"

[project]
name = "pre_d"
requires-python = ">=3.8"
classifiers = [
    "Programming Language :: Rust",
    "Programming Language :: Python :: Implementation :: CPython",
]
Use code with caution.

Cargo.toml

This file instructs Cargo to compile the library as a C-compatible shared object (cdylib), which Python can load.
toml
[package]
name = "pre_d"
version = "0.1.0"
edition = "2021"

[lib]
name = "pre_d"
crate-type = ["cdylib"]

[dependencies]
pyo3 = { version = "0.22", features = ["extension-module"] }
Use code with caution.

src/lib.rs

Place your modern PyO3 source code here:
rust
use pyo3::prelude::*;

const PHI: f64 = 1.618033988749895;
const NODES: usize = 121;

#[pyclass]
struct SamaranKernel {
    glyph_weights: Vec<f64>,
}

#[pymethods]
impl SamaranKernel {
    #[new]
    fn new() -> Self {
        let weights = (0..NODES)
            .map(|i| (i as f64 * PHI).sin().abs())
            .collect();
        Self { glyph_weights: weights }
    }

    fn pre_determine(&self, input_signal: Vec<f64>) -> PyResult<f64> {
        if input_signal.is_empty() {
            return Ok(0.0);
        }

        let resonance: f64 = input_signal
            .iter()
            .enumerate()
            .map(|(i, &val)| {
                let node_index = i % NODES;
                val * self.glyph_weights[node_index] * PHI
            })
            .sum();

        Ok(resonance / (input_signal.len() as f64))
    }
}

#[pymodule]
fn pre_d(m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_class::<SamaranKernel>()?;
    Ok(())
}
Use code with caution.

3. Build & Installation Steps

Run these commands inside your terminal from the root my_project/ directory:
1. Install Maturin into your Python environment:bash
pip install maturin
Use code with caution.
2. Compile and build the extension locally for development:bash
maturin develop
Use code with caution.
(This instantly builds the Rust package and installs the pre_d module into your current virtual environment.)

4. Verification Script

You can verify it works by opening a Python shell or creating a quick script:
python
from pre_d import SamaranKernel

# Initialize the kernel
kernel = SamaranKernel()

# Create a sample signal
signal = [1.0, 2.5, -0.5, 4.2]

# Compute resonance
result = kernel.pre_determine(signal)
print(f"Resonance result: {result}")
Use code with caution.
If you run into any platform-specific toolchain issues (such as missing target linkages, dynamic compiler dependencies, or library issues during maturin develop), let me know your operating system and current terminal error so we can patch it.
