project(
    'ppl',
    'cpp',
    version: '1.2',
)

# ---------- Build PPL using its native autotools build system ----------
#
# Instead of reimplementing the entire PPL build in Meson, we invoke the
# upstream autotools configure & make.  This mirrors the approach used by
# primecountpy (method = cmake) – except PPL ships autotools, so we call
# ./configure + make here.

gmp_dep = dependency('gmp')
gmpxx_dep = dependency('gmpxx')

# Locate GMP include/lib dirs for autotools --with-gmp-include / --with-gmp-lib.
# On multiarch systems (e.g. Debian/Ubuntu) headers may live under
# /usr/include/<triplet> while libs are under /usr/lib/<triplet>.
# We find the actual directory containing gmp.h and the directory with libgmp.
gmp_libdir = run_command(
    'pkg-config', '--variable=libdir', 'gmp', check: true,
).stdout().strip()
# Ask the preprocessor where it finds gmp.h (handles multiarch reliably).
gmp_includedir = run_command(
    'sh', '-c',
    '''echo '#include <gmp.h>' | cpp -M - 2>/dev/null | tr ' ' '\n' | grep '/gmp\\.h$' | head -1 | xargs dirname''',
    check: false,
).stdout().strip()

# GLPK (optional) – PPL auto-detects glpk.h; we pass CPPFLAGS/LDFLAGS so
# configure can find it even in non-standard locations.
cpp = meson.get_compiler('cpp')
glpk_dep = dependency('glpk', required: false)
if not glpk_dep.found()
    # No pkg-config file; try to detect glpk.h directly.
    if cpp.has_header('glpk.h')
        glpk_dep = declare_dependency(link_args: ['-lglpk'])
    endif
endif
# Locate GLPK paths for configure's header/library search.
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

nproc_result = run_command('nproc', check: false)
nproc = nproc_result.returncode() == 0 ? nproc_result.stdout().strip() : '1'

configure_flags = [
    '--prefix="' + ppl_install_dir + '"',
    '--disable-shared',
    '--enable-static',
    '--disable-documentation',
]
if gmp_includedir != ''
    configure_flags += ['--with-gmp-include=' + gmp_includedir]
endif
if gmp_libdir != ''
    configure_flags += ['--with-gmp-lib=' + gmp_libdir]
endif

# Build CPPFLAGS/LDFLAGS to help configure find GLPK (no --with-glpk in PPL 1.2).
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

# Run ./configure && make && make install at configure time.
# The tarball already ships a pre-generated configure script.
run_command(
    'sh', '-c',
    ' '.join([
        'set -e &&',
        'mkdir -p "' + ppl_build_dir + '" &&',
        'cd "' + ppl_build_dir + '" &&',
        '"' + ppl_source_dir + '/configure"',
    ] + configure_flags + [
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
all_deps = [gmp_dep, gmpxx_dep]
if glpk_dep.found()
    all_deps += [glpk_dep]
endif
ppl_dep = declare_dependency(
    compile_args: [
        '-I' + ppl_install_dir / 'include',
    ],
    link_args: [ppl_install_dir / 'lib' / 'libppl.a'],
    dependencies: all_deps,
)
