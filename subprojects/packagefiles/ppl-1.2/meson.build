project(
    'ppl',
    'cpp',
    version: '1.2',
    meson_version: '>=0.63.0',
)

mod = import('unstable-external_project')

configure_options = [
    '--disable-shared',
    '--enable-static',
    '--disable-documentation',
]

# PPL needs GMP; if it comes from a subproject, tell configure where it is.
gmp_dep = dependency('gmp')
gmp_prefix = gmp_dep.get_variable(internal: 'prefix', default_value: '')
if gmp_prefix != ''
    configure_options += ['--with-gmp=' + gmp_prefix]
endif

# PPL can optionally use GLPK; if available, tell configure where it is.
glpk_dep = dependency('glpk', required: false)
if glpk_dep.found()
    glpk_prefix = glpk_dep.get_variable(internal: 'prefix', default_value: '')
    if glpk_prefix != ''
        configure_options += ['--with-glpk=' + glpk_prefix]
    endif
endif

p = mod.add_project(
    'configure',
    configure_options: configure_options,
    env: ['CXXFLAGS=-DNDEBUG -O2 -fPIC'],
)

ppl_dep = p.dependency('ppl')
meson.override_dependency('ppl', ppl_dep)
