project(
	'dwl',
	'c',
	version: run_command('git', 'describe', '--tags', '--dirty', check: false).stdout().strip(),
	license: [ 'GPL-3.0-only', 'CC0-1.0', 'MIT' ],
	meson_version: '>=1.3',
	default_options: [
		'c_std=c99',
		'warning_level=2',
	],
)

configure_file(
	input: 'config.def.h',
	output: 'config.h',
	copy: true,
	install_dir: '.',
)

cc = meson.get_compiler('c')

add_project_arguments([
	'-DWLR_USE_UNSTABLE',
	'-D_POSIX_C_SOURCE=200809L',
	'-DVERSION="@0@"'.format(meson.project_version()),

	'-Wno-unused-parameter',
], language: 'c')

wlroots = subproject('wlroots',
	default_options: [
		'backends=drm,libinput,auto',
		'default_library=static',
		'examples=false',
		'session=enabled',
	],
)
wlroots_has_xwlr = wlroots.get_variable('features').get('xwayland')

libinput = dependency('libinput')
math = cc.find_library('m')
wayland_server = dependency('wayland-server')
xcb = dependency('xcb', required: wlroots_has_xwlr)
xcb_icccm = dependency('xcb-icccm', required: wlroots_has_xwlr)
xkbcommon = dependency('xkbcommon')

dwl_deps = [
	libinput,
	math,
	wayland_server,
	wlroots.get_variable('wlroots'),
	xkbcommon,
]

if wlroots_has_xwlr
	add_project_arguments('-DXWAYLAND', language: 'c')
	dwl_deps += [ xcb, xcb_icccm ]
endif

wayland_protos = dependency('wayland-protocols')
wayland_scanner = dependency('wayland-scanner')
wayland_protocol_dir = wayland_protos.get_variable('pkgdatadir')
wayland_scanner_prog = find_program(
	wayland_scanner.get_variable('wayland_scanner'),
	native: true,
)

protocols = [
	wayland_protocol_dir / 'staging/cursor-shape/cursor-shape-v1.xml',
	wayland_protocol_dir / 'unstable/pointer-constraints/pointer-constraints-unstable-v1.xml',
	'protocols/wlr-layer-shell-unstable-v1.xml',
	'protocols/wlr-output-power-management-unstable-v1.xml',
	wayland_protocol_dir / 'stable/xdg-shell/xdg-shell.xml',
]
protocols_src = []

wayland_scanner_server = generator(
	wayland_scanner_prog,
	output: '@BASENAME@-protocol.h',
	arguments: ['server-header', '@INPUT@', '@OUTPUT@'],
)

foreach xml : protocols
	protocols_src += wayland_scanner_server.process(xml)
endforeach

executable(
	'dwl',
	[ 'dwl.c', 'util.c', protocols_src ],
	include_directories: [include_directories('.')],
	dependencies: dwl_deps,
	install: true,
)
