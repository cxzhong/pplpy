project('glpk', 'c', version: '5.0')

fs = import('fs')

src_dir = meson.project_source_root()
build_dir = meson.project_build_root() / 'build'
install_dir = meson.project_build_root() / 'install'

if not fs.exists(install_dir / 'lib' / 'libglpk.a')
    message('Building GLPK from source...')
    run_command(
        'bash', '-c',
        'mkdir -p "@0@" && cd "@0@" && "@1@/configure" --disable-shared --enable-static --with-pic --prefix="@2@" CFLAGS="-O2 -fPIC" && make -j$(nproc 2>/dev/null || sysctl -n hw.logicalcpu 2>/dev/null || echo 4) && make install'.format(
            build_dir, src_dir, install_dir,
        ),
        check: true,
    )
endif

glpk_dep = declare_dependency(
    compile_args: ['-I' + install_dir / 'include'],
    link_args: ['-L' + install_dir / 'lib', '-lglpk'],
    variables: {'prefix': install_dir},
)

meson.override_dependency('glpk', glpk_dep)
