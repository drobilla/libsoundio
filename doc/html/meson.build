# Copyright 2022 David Robillard <d@drobilla.net>
# SPDX-License-Identifier: 0BSD OR MIT

index_html = custom_target(
  'index.html',
  command: [doxygen, '@INPUT0@'],
  input: [doxygen_conf] + config_h,
  output: 'index.html',
  build_by_default: true,
)

