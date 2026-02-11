project(
    'ppl',
    'cpp',
    version: '1.2',
)

# ---------- Build PPL using its native autotools build system ----------

gmp_dep = dependency('gmp')
gmpxx_dep = dependency('gmpxx')

ppl_source_dir = meson.current_source_dir()
ppl_build_dir = meson.current_build_dir() / '_autotools'
ppl_install_dir = meson.current_build_dir() / '_install'

# nproc is Linux-only; use Python for cross-platform CPU count.
nproc_result = run_command(
    'python3',
    '-c',
    'import os; print(os.cpu_count() or 1)',
    check: false,
)
nproc = nproc_result.returncode() == 0 ? nproc_result.stdout().strip() : '1'

# ---------- Collect flags from pkg-config ----------
# pkg-config --cflags / --libs-only-L already handles multiarch and
# non-standard prefix locations; no need to resolve paths manually.
gmp_cflags = run_command(
    'pkg-config', '--cflags', 'gmp', 'gmpxx',
    check: true,
).stdout().strip()
gmp_ldflags = run_command(
    'pkg-config', '--libs-only-L', 'gmp', 'gmpxx',
    check: true,
).stdout().strip()

# GLPK is optional; ignore errors if not installed.
glpk_cflags = run_command(
    'sh', '-c', 'pkg-config --cflags glpk 2>/dev/null || true',
    check: false,
).stdout().strip()
glpk_ldflags = run_command(
    'sh', '-c', 'pkg-config --libs-only-L glpk 2>/dev/null || true',
    check: false,
).stdout().strip()

extra_cppflags = []
extra_ldflags = []
if gmp_cflags != ''
    extra_cppflags += [gmp_cflags]
endif
if glpk_cflags != ''
    extra_cppflags += [glpk_cflags]
endif
if gmp_ldflags != ''
    extra_ldflags += [gmp_ldflags]
endif
if glpk_ldflags != ''
    extra_ldflags += [glpk_ldflags]
endif

# ---------- Configure options ----------
configure_options = [
    '--prefix="' + ppl_install_dir + '"',
    '--disable-shared',
    '--enable-static',
    '--disable-documentation',
]

configure_env = ['CXXFLAGS="-DNDEBUG -O2 -fPIC"']
if extra_cppflags.length() > 0
    configure_env += ['CPPFLAGS="' + ' '.join(extra_cppflags) + '"']
endif
if extra_ldflags.length() > 0
    configure_env += ['LDFLAGS="' + ' '.join(extra_ldflags) + '"']
endif

# ---------- Build PPL via autotools ----------
run_command(
    'sh',
    '-c',
    ' '.join(
        [
            'set -e &&',
            'mkdir -p "' + ppl_build_dir + '" &&',
            'cd "' + ppl_build_dir + '" &&',
            '"' + ppl_source_dir + '/configure"',
        ] + configure_options + configure_env + [
            '&&',
            'make -j' + nproc,
            '&&',
            'make install',
        ],
    ),
    check: true,
)

# Expose the autotools-built library as a Meson dependency.
ppl_dep = declare_dependency(
    compile_args: ['-I' + ppl_install_dir / 'include'],
    link_args: [ppl_install_dir / 'lib' / 'libppl.a'],
    dependencies: [gmp_dep, gmpxx_dep],
)
