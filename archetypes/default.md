+++
date         = '{{ .Date }}'
draft        = true
title        = '{{ replace .File.ContentBaseName "-" " " | title }}'
description  = 'Colibri allows you to deploy Large Language Models (LLMs) on your system, whether it is a small GPU or even on the CPU.'
searchHidden = false
comments     = false
TocOpen      = true

[cover]
#  image = "cover.png"
#  alt   = "Joule Studio Developer Certification"

[ai]
  level        = 'assisted' #assisted or generated
  note         = 'Optimized the Post, and its formatting'
  model        = 'GLM 5.3 via Mistral AI'
  hiddenInList = true
+++
