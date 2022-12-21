# Copyright 2022 David Robillard <d@drobilla.net>
# SPDX-License-Identifier: 0BSD OR MIT

doc_conf_data = configuration_data()
doc_conf_data.set('CMAKE_BINARY_DIR', meson.project_build_root())
doc_conf_data.set('CMAKE_SOURCE_DIR', soundio_src_root)
doc_conf_data.set('CONFIGURE_OUT_FILE', meson.project_build_root() / 'config.h')
doc_conf_data.set('LIBSOUNDIO_VERSION', meson.project_version())

doxygen_conf = configure_file(
  input: files('doxygen.conf.in'),
  output: 'doxygen.conf',
  configuration: doc_conf_data,
)

subdir('html')
