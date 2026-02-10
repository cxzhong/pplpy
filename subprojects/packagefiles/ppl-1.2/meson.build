project(
    'ppl',
    'cpp',
    version: '1.2',
)

# ---------- Build PPL using its native autotools build system ----------
#
# We invoke configure-wrapper (which runs the real ./configure and
# generates a ppl.pc pkg-config file) followed by make && make install
# at Meson configure time.
#
# The unstable-external_project module (used by sagemath/sage's ECL
# subproject) cannot be used here because it registers PPL's install
# targets with Meson, and meson-python cannot map those to wheel paths.

gmp_dep = dependency('gmp')
gmpxx_dep = dependency('gmpxx')

# ---------- Locate GMP for --with-gmp-include / --with-gmp-lib ----------
# On multiarch systems (e.g. Debian/Ubuntu) headers may live under
# /usr/include/<triplet> while libs are under /usr/lib/<triplet>.
gmp_libdir = run_command(
    'pkg-config', '--variable=libdir', 'gmp', check: true,
).stdout().strip()
# Ask the preprocessor where it finds gmp.h (handles multiarch reliably).
gmp_includedir = run_command(
    'sh', '-c',
    '''echo '#include <gmp.h>' | cpp -M - 2>/dev/null | tr ' ' '\n' | grep '/gmp\\.h$' | head -1 | xargs dirname''',
    check: false,
).stdout().strip()

# ---------- Locate GLPK (optional) ----------
# PPL auto-detects glpk.h; pass paths via CPPFLAGS/LDFLAGS so configure
# can find it even in non-standard locations.
glpk_includedir = run_command(
    'sh', '-c',
    '''echo '#include <glpk.h>' | cpp -M - 2>/dev/null | tr ' ' '\n' | grep '/glpk\\.h$' | head -1 | xargs dirname''',
    check: false,
).stdout().strip()
glpk_libdir = run_command(
    'sh', '-c',
    'pkg-config --variable=libdir glpk 2>/dev/null || echo ""',
    check: false,
).stdout().strip()

ppl_source_dir = meson.current_source_dir()
ppl_build_dir = meson.current_build_dir() / '_autotools'
ppl_install_dir = meson.current_build_dir() / '_install'

# nproc is Linux-only; use Python for cross-platform CPU count.
nproc_result = run_command(
    'python3', '-c', 'import os; print(os.cpu_count() or 1)', check: false,
)
nproc = nproc_result.returncode() == 0 ? nproc_result.stdout().strip() : '1'

# ---------- Configure options ----------
configure_options = [
    '--prefix="' + ppl_install_dir + '"',
    '--disable-shared',
    '--enable-static',
    '--disable-documentation',
]
if gmp_includedir != ''
    configure_options += ['--with-gmp-include=' + gmp_includedir]
endif
if gmp_libdir != ''
    configure_options += ['--with-gmp-lib=' + gmp_libdir]
endif

# ---------- Environment variables ----------
# GLPK: no --with-glpk flag in PPL 1.2; pass paths via CPPFLAGS/LDFLAGS.
extra_cppflags = []
extra_ldflags = []
if glpk_includedir != ''
    extra_cppflags += ['-I' + glpk_includedir]
endif
if glpk_libdir != ''
    extra_ldflags += ['-L' + glpk_libdir]
endif

configure_env = []
if extra_cppflags.length() > 0
    configure_env += ['CPPFLAGS="' + ' '.join(extra_cppflags) + '"']
endif
if extra_ldflags.length() > 0
    configure_env += ['LDFLAGS="' + ' '.join(extra_ldflags) + '"']
endif

# ---------- Build PPL via autotools ----------
run_command(
    'sh', '-c',
    ' '.join([
        'set -e &&',
        'mkdir -p "' + ppl_build_dir + '" &&',
        'cd "' + ppl_build_dir + '" &&',
        '"' + ppl_source_dir + '/configure"',
    ] + configure_options + [
        'CXXFLAGS="-DNDEBUG -O2 -fPIC"',
    ] + configure_env + [
        '&&',
        'make -j' + nproc,
        '&&',
        'make install',
    ]),
    check: true,
)

# Expose the autotools-built library as a Meson dependency.
ppl_dep = declare_dependency(
    compile_args: ['-I' + ppl_install_dir / 'include'],
    link_args: [ppl_install_dir / 'lib' / 'libppl.a'],
    dependencies: [gmp_dep, gmpxx_dep],
)
