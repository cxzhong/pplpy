# ---------- Dependencies ----------
gmp_dep = dependency('gmp', required: false)
cpp = meson.get_compiler('cpp')
if not gmp_dep.found()
  gmp_dep = cpp.find_library('gmp', static: true, required: false)
  if not gmp_dep.found()
    gmp_dep = cpp.find_library('gmp')
  endif
endif
gmpxx_dep = dependency('gmpxx', required: false)
if not gmpxx_dep.found()
  gmpxx_dep = cpp.find_library('gmpxx', static: true, required: false)
  if not gmpxx_dep.found()
    gmpxx_dep = cpp.find_library('gmpxx')
  endif
endif

cysignals_dep = dependency('cysignals', required: false)
if not cysignals_dep.found()
  # cysignals installs headers alongside the Python package
  cysignals_inc = run_command(py, '-c',
    'import cysignals; import os; print(os.path.dirname(cysignals.__file__))',
    check: true,
  ).stdout().strip()
  cysignals_dep = declare_dependency(
    compile_args: ['-I' + cysignals_inc],
  )
endif

gmpy2_dep = dependency('gmpy2', required: false)
if not gmpy2_dep.found()
  gmpy2_inc = run_command(py, '-c',
    'import gmpy2; print(gmpy2.__file__.rsplit("/",1)[0])',
    check: true,
  ).stdout().strip()
  gmpy2_dep = declare_dependency(
    compile_args: ['-I' + gmpy2_inc],
  )
endif

# Include directories for ppl_shim.hh
ppl_shim_inc = include_directories('.')

# Common dependencies for all Cython extensions
common_deps = [ppl_dep, gmp_dep, gmpxx_dep, cysignals_dep, gmpy2_dep]
common_cpp_args = ['-DNDEBUG']

# ---------- Cython extension modules ----------
# Each module needs:
#   - The .pyx file  (Cython transpiles it to .cpp)
#   - ppl_shim.cc    (small C++ helper compiled alongside)

extensions = {
  'linear_algebra': 'linear_algebra.pyx',
  'mip_problem':    'mip_problem.pyx',
  'polyhedron':     'polyhedron.pyx',
  'generator':      'generator.pyx',
  'constraint':     'constraint.pyx',
  'congruence':     'congruence.pyx',
  'bit_arrays':     'bit_arrays.pyx',
}

foreach ext_name, pyx_file : extensions
  py.extension_module(ext_name,
    pyx_file, 'ppl_shim.cc',
    dependencies: common_deps,
    include_directories: ppl_shim_inc,
    override_options: ['cython_language=cpp'],
    install: true,
    install_dir: py.get_install_dir() / 'ppl',
    cpp_args: common_cpp_args,
    cython_args: [
      '--directive', 'embedsignature=True',
      '--directive', 'language_level=3',
    ],
  )
endforeach

# ---------- Install pure-Python and data files ----------
py.install_sources(
  '__init__.py',
  subdir: 'ppl',
)

# Copy __init__.py into builddir/ppl/ so tests can import ppl from the build tree
configure_file(input: '__init__.py', output: '__init__.py', copy: true)

# Install .pxd files so downstream Cython code can cimport ppl
py.install_sources(
  'ppl_decl.pxd',
  'linear_algebra.pxd',
  'mip_problem.pxd',
  'polyhedron.pxd',
  'generator.pxd',
  'constraint.pxd',
  'congruence.pxd',
  'bit_arrays.pxd',
  subdir: 'ppl',
)

# Install C++ shim headers
install_data(
  'ppl_shim.hh',
  install_dir: py.get_install_dir() / 'ppl',
)
